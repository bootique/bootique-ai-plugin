# Bootique 4.0 Overview

This is a working reference for the parts of Bootique that matter when discovering configuration and running apps. It is not a full manual — it is the model the skills in this plugin operate on.

## Forget the Old Bootique Behavior

Forget the old Bootique behavior, no Longer Relevant in 4.0 or earlier

* Older Bootique versions used Guice for dependency injection. The current ones (2.0+) rely on Bootique own DI engine. No Guice should ever be mentioned or considered
* Very early Bootique versions used naming conventions for environment variables. E.g. `BQ_FOO_URL` would auto-bind to the "foo.url" property. Those do not work for a long long time, and should not be offered or mentioned. 


## The shape of a Bootique app

A Bootique app is a runnable Java jar whose `main` calls `Bootique.app(args).autoLoadModules().exec()` (or a close variant). At runtime, Bootique:

1. Loads modules — both those declared explicitly and those auto-loaded via `META-INF/services/io.bootique.BQModule`.
2. Each module's `crate()` method returns a `ModuleCrate` that describes module name, purpose, and most importantly, configuration (config roots). 
3. The rest of the module methods declare and bind services, commands, etc. (via methods annotated with `@Provides`, and  `configure(Binder)` method)
3. Bootique parses CLI args, merges YAML config (from `--config=<path>` and any built-in classpath defaults) with environment variables, and then dispatches to a command.

Two facts that fall out of this and matter for the skills:

- **There is no single "config schema file."** The schema is the union of every config root contributed by every loaded module. To know what a given app accepts, you need to ask the runtime, or reconstruct the union by walking modules.
- **The runnable jar is authoritative.** Whatever modules end up on its classpath is exactly what the app exposes. `--help-config` reflects that exact set; static analysis can drift if a module is on the dependency tree but not actually loaded.

## Module structure (4.0)

Each module overrides `crate()`:

```java
public class FooModule implements BQModule {
    @Override
    public ModuleCrate crate() {
        return ModuleCrate.of(this)
                .description("Foo support for Bootique")
                .config("foo", FooFactory.class)
                .build();
    }

    @Override
    public void configure(Binder binder) {
        // Bootique DI bindings; not relevant to config discovery
    }


    // Bootique DI bindings; not relevant to config discovery
    @Provides
    MyService provideService(MyOtherService dep) {
        return new MyService(dep);
    }

}
```

`ModuleCrate.config(prefix, FactoryClass.class)` is the line that registers a **config root**. The string `"foo"` becomes the top-level YAML key; `FooFactory.class` is the factory bound to it.

A module may register multiple roots, contribute commands, or contribute to existing roots.

## Factory classes: `@BQConfig` and `@BQConfigProperty`

A factory is a plain Java class whose **setters** are the configurable knobs. Setters are marked with `@BQConfigProperty`:

```java
@BQConfig("Connects to a relational database")
public class DataSourceFactory {

    @BQConfigProperty("JDBC URL")
    public void setUrl(String url) { ... }

    @BQConfigProperty("Connection pool size")
    public void setPoolSize(int poolSize) { ... }

    @BQConfigProperty
    public void setRetry(RetryFactory retry) { ... }
}
```

Rules used by both discovery paths:

- A setter annotated `@BQConfigProperty` is a configurable property. Its name is derived from the setter (`setPoolSize` → `poolSize`).
- The property type determines the YAML shape:
  - **Primitive / wrapper / `String` / `Duration` / `URL` / enum** → leaf value.
  - **`List<X>` / `Map<String, X>`** → collection; recurse into `X` if it is itself a `@BQConfig` class.
  - **A class annotated `@BQConfig`** → a non-leaf node; recurse into its `@BQConfigProperty` setters.
  - **Polymorphic types** (`@JsonTypeInfo` + `@BQConfig`) → recurse into each known subtype; YAML uses a `type:` discriminator. Polymorphic types when present, are listed in `META-INF/services/io.bootique.config.PolymorphicConfiguration` in a corresponding module

## CLI flags worth knowing

Bootique apps share a small set of standard flags. Custom commands add their own.

| Flag | What it does |
|---|---|
| `--config=<path>` | Layered YAML config; can be repeated, later files override earlier ones. URL form (`http://`, `classpath:`) accepted. |
| `--help`, `-h` | Print available commands and options. |
| `--help-config`, `-H`| Print the **full effective config tree** as commented YAML, derived from every loaded module's config roots. This is the gold-standard discovery source. |
| `--exec` | Execute one or more named jobs. Pair with `-j <jobName>` (repeatable) to select them. Provided by `bootique-job`, and may be absent|
| `--server` | (If `bootique-jetty` is on the classpath) Start an embedded Jetty server. Provided by `bootique-jetty` , and may be absent. |
| `--lb-update`, `-u` |Run DB migration scripts. Provided by `bootique-liquibase`, and may be absent |


If a command isn't listed here, run `--help` against the app and read what's there. Don't assume — modules contribute commands freely.

## YAML, env vars, and overrides

Config sources, in order of precedence (later wins):

1. Module-provided defaults (whatever the factory's no-arg constructor and field initializers establish).
2. Each `--config=<path>` in left-to-right order.
3. Environment variables bound to config paths.
4. `-D` system properties following config paths (rare, not recommended!!)

**Environment variable mapping.** Bootique does not auto-derive env var names from config paths. A binding has to be declared, typically inside a module's `configure(Binder)` via `BQCoreModule.extend(binder).declareVar("foo.url", "MYAPP_FOO_URL")`. So when a user asks "what env var sets `foo.url`?", the answer requires reading the relevant module's binder code.

If no declared mapping exists, the user has 2 options:
- Add a `declareVar` binding in a module they own.
- Edit the YAML.

## What this means for the skills

- Prefer **`--help-config`** for discovery. It is fast, complete, and authoritative.
- Fall back to **static analysis** (walking `ModuleCrate.config(...)` → factory → `@BQConfigProperty`) only when running the app isn't viable (no built jar, build fails, etc.).
- When telling the user how to override a value, **check for declared env var bindings** before suggesting one.
