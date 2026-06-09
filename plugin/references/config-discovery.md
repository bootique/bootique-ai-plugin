# Discovering the Configuration Tree

There are two ways to learn what a Bootique app accepts as configuration. Use them in this order.

## Path 1 — `--help-config` (preferred)

The runtime knows the truth. `--help-config` prints the merged YAML schema for every config root every loaded module contributes, with descriptions pulled from `@BQConfig` / `@BQConfigProperty` annotations.

```bash
java -jar <module>/target/<finalName>.jar --help-config
```

The output is commented YAML — top-level keys are config roots, indented blocks describe nested factories. Treat it as the source of truth and stop here unless something forces you to fall back.

**When the runnable jar isn't ready**, the build step is part of discovery:

```bash
mvn -pl <module-path> -am install -DskipTests \
  && java -jar <module>/target/<finalName>.jar --help-config
```

Cache the output for the session — running the JVM repeatedly is slow.

### When `--help-config` is unavailable

A few situations force a fallback:

- The project doesn't build (compile errors the user is mid-fixing).
- The runtime fails before `--help-config` prints (a module's `configure(Binder)` throws).
- The user explicitly asks "without running anything."
- You only have access to source, not a built jar.

In each case, fall through to Path 2.

## Path 2 — Static analysis

Reconstruct the config tree by walking modules.

### Step A — Enumerate the loaded modules

A module is loaded if either:

1. It is registered as a service in `META-INF/services/io.bootique.BQModule` on the classpath of the runnable jar, **or**
2. It is added explicitly via `Bootique.app(args).module(SomeModule.class)` in the app's `main`.

To get the union, do both:

```bash
# Service-loaded modules across all runtime dependencies of the runnable module
find . -path '*/META-INF/services/io.bootique.BQModule' -not -path '*/target/*' -exec cat {} \;

# Explicitly-added modules in the runnable module's main class(es)
grep -rn "\.module(" --include='*.java' <runnable-module>/src/main/java
```

If you only have the source tree (not built classes), `META-INF/services/...` files live under each module's `src/main/resources/META-INF/services/`.

### Step B — Find each module's config roots

For every module class, locate its `crate()` method and read the `.config(prefix, FactoryClass.class)` calls. Each one is a top-level YAML key bound to a factory.

```bash
# For each module class found above:
grep -A 20 "ModuleCrate crate" <module-class>.java | grep "\.config("
```

A single module may register zero, one, or several config roots. Modules without `.config(...)` calls don't contribute to the YAML schema.

### Step C — Walk each factory

For every factory class, the configurable surface is the set of `@BQConfigProperty`-annotated setters. For each setter:

1. Derive the property name from the setter (`setPoolSize` → `poolSize`).
2. Look at the parameter type:
   - **Primitive / wrapper / `String` / `Duration` / `URL` / enum** → leaf value. Done.
   - **`List<X>` / `Set<X>`** → YAML list. If `X` has `@BQConfig`, recurse into it.
   - **`Map<String, X>`** → YAML map of named entries. If `X` has `@BQConfig`, recurse into it (each entry uses the map key as its name).
   - **A class annotated `@BQConfig`** → non-leaf node. Recurse into its `@BQConfigProperty` setters.
   - **Polymorphic** (`@JsonTypeInfo` + abstract class with `@BQConfig`) → enumerate concrete subtypes; each YAML entry uses a `type:` discriminator.
3. The `@BQConfig` and `@BQConfigProperty` `value()` strings are the human descriptions — surface them when reporting.

### Step D — Watch for traps

- **Missing `@BQConfig` on a class** while its setters have `@BQConfigProperty` is a likely bug in the target codebase, not a discovery error. Surface it to the user rather than silently traversing.
- **Inherited setters** count. If `BarFactory extends FooFactory`, the inherited `@BQConfigProperty` setters are part of `BarFactory`'s schema.

## Reporting the tree to the user

Match the user's level of detail. If they asked "what config does this app accept?" give the YAML skeleton with a one-line description per node. If they asked "where do I set the JDBC URL?" find the single property and answer with the YAML path plus the factory class file:line.
