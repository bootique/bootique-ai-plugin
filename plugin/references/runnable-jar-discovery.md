# Finding the Runnable Jar (Maven)

Both skills need to know **which Maven module produces the runnable jar** and **where the jar lands after a build**. This document covers both.

## Why this matters

In a multi-module repo, only one (or a few) modules build into a runnable jar. The others are libraries. You cannot run `--help-config` against a library. You also cannot guess by module name — `app`, `service`, `runner`, `main`, `cli`, `<project>-app` are all common, and so are project-specific names.

## Two recipes, same outcome

Bootique apps typically use one of two packaging recipes:

### Recipe A — Runnable Jar with `lib/` Folder

The build produces `target/<finalName>.jar` containing only the app's own classes, plus a `target/lib/` directory full of dependency jars. The jar's manifest sets `Class-Path: lib/...` so `java -jar` picks them up.

Telltale POM signs:
- `maven-jar-plugin` configured with `addClasspath=true` and `classpathPrefix=lib/`.
- `maven-dependency-plugin` `copy-dependencies` goal targeting `${project.build.directory}/lib`.

To run: `java -jar target/<finalName>.jar` from the module directory (the `lib/` folder must be next to the jar).

### Recipe B — Runnable Jar with Dependencies (uberjar)

The build produces a single `target/<finalName>.jar` with all dependency classes shaded in.

Telltale POM signs:
- `maven-shade-plugin` with `shadedArtifactAttached=false` (or true with a classifier — read it).
- A `<transformer>` for the manifest that sets `Main-Class`.

To run: `java -jar target/<finalName>.jar` from anywhere — the jar is self-contained.

## Discovery procedure

Run these from the repo root in order. Stop at the first one that yields a clear answer.

### 1. Check for a runnable-jar module via `pom.xml` patterns

```bash
# Modules using maven-shade-plugin (Recipe B — uberjar)
grep -l "maven-shade-plugin" $(find . -name pom.xml -not -path '*/target/*')

# Modules using copy-dependencies into lib/ (Recipe A — lib folder)
grep -l "copy-dependencies" $(find . -name pom.xml -not -path '*/target/*')
```

Either list narrows you to candidate modules. Often there's exactly one.

### 2. Confirm via `Main-Class`

A runnable Bootique module has a class with a `main` that calls `Bootique.app(args)`. Confirm with:

```bash
grep -rn "Bootique.app(" --include='*.java' <candidate-module>/src/main/java
```

or

```bash
grep -rn "Bootique.main(" --include='*.java' <candidate-module>/src/main/java
```

If the candidate has it, that's your runnable module. If not, keep looking — sometimes the main class is in a different module than the one with the shading config (uncommon but possible).

### 3. Locate the built jar

After `mvn verify` (or just `mvn package` on the runnable module):

```bash
# Recipe A — jar plus lib/
ls <module>/target/*.jar <module>/target/lib/ 2>/dev/null

# Recipe B — uberjar (no lib/)
ls <module>/target/*.jar 2>/dev/null
```

The jar's `<finalName>` defaults to `${project.artifactId}-${project.version}` but is sometimes customized in the POM (`<build><finalName>...`).

### 4. If no jar exists yet

Build only what's needed:

```bash
# From the runnable module:
mvn -pl . -am verify -DskipTests
```

`-pl .` says "this module," `-am` says "and dependencies." `-DskipTests` because we're trying to get a runnable artifact, not validate behavior. If the user's intent involves correctness (e.g., running a job for real), drop `-DskipTests`.

If the project uses the reactor build from the root, this also works:

```bash
mvn -pl <module-path> -am verify -DskipTests
```

## Multiple runnable modules

Some repos ship more than one runnable jar (e.g., a web app and a migrations app). When ambiguous, ask the user which one they mean, and remember the answer for the rest of the session.

## Caching the result

Once located, treat the runnable jar's path as stable for the session. Re-discovery on every invocation is wasteful and sometimes user-confusing. If the user changes context (different repo, different module), redo discovery.
