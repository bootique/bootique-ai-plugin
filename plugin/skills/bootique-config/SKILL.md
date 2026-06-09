---
name: bootique-config
description: "Use this skill whenever the user wants to inspect, create, or modify the configuration of a Bootique 4.0 Java application — including YAML config files, environment-variable / system-property overrides, or `@BQConfig`-annotated factory classes. Trigger on phrases like 'add a setting to my Bootique app', 'change the config', 'where does this app set X', 'expose this as configurable', 'what does this app accept in YAML', 'dump the config schema', or any mention of a Bootique YAML file (`*.yml` near a Bootique app) or a factory class with `@BQConfig` / `@BQConfigProperty`. Also trigger when the user references config concepts (URL, pool size, timeouts, jobs, datasources) in the context of a Bootique app, even without saying 'configure'. The skill discovers the effective config tree by running `--help-config` against the built jar when possible, falls back to static analysis of `BQModule.crate()` config roots and `@BQConfig` factories, and then either makes the requested edit or explains the structure."
---

# bootique-config

Inspect or change the configuration surface of a Bootique 4.0 Java app. Three sub-flows: **discover**, **edit YAML**, **edit factory class**.

## Required reading

Before doing anything, read these — they describe the model this skill operates on:

- `${CLAUDE_PLUGIN_ROOT}/references/bootique-overview.md`
- `${CLAUDE_PLUGIN_ROOT}/references/runnable-jar-discovery.md`
- `${CLAUDE_PLUGIN_ROOT}/references/config-discovery.md`

## Step 1 — Locate the runnable module

Configuration is per-app, and "the app" is the runnable jar's module. Follow `runnable-jar-discovery.md` to find it. If multiple runnable modules exist and the user's request is ambiguous, ask which one. Cache the answer for the rest of the session.

## Step 2 — Discover the config tree

Follow `config-discovery.md`:

1. Try `--help-config` against the built jar. If the jar isn't there, build it (`mvn -pl <module> -am install -DskipTests`).
2. If `--help-config` is unavailable for any reason, fall back to static analysis: walk `ModuleCrate.config(...)` roots → factory classes → `@BQConfigProperty` setters.

Cache the result. Don't re-run `--help-config` for follow-up questions in the same session.

## Step 3 — Match the request to a sub-flow

Read what the user actually wants. Common shapes:

| User says | Sub-flow |
|---|---|
| "What config does this app accept?" / "Show me the YAML schema" | **Discover-only** — print the relevant slice of the tree with descriptions. |
| "Add `foo.bar.timeout: 30s` to the dev config" / "Change the JDBC URL" | **Edit YAML** |
| "I want to expose `retryCount` as a configurable property" / "Make this setting configurable from YAML" | **Edit factory class** |
| "Override `foo.bar` without editing the file" | **Override** — system property or declared env var (see overview). |

If the request is ambiguous, ask one clarifying question. Don't guess between editing YAML and editing Java.

## Sub-flow A — Discover-only

Show the user the relevant part of the tree. Don't dump the entire `--help-config` output unless they asked for it — find the specific node they care about and report:

- YAML path (e.g., `jdbc.myDS.url`)
- Type (string, int, `Duration`, etc.)
- Description from `@BQConfigProperty("...")`
- Factory class:line where the setter lives, so the user can jump in

If a property they asked about doesn't exist, say so plainly. Don't invent one.

## Sub-flow B — Edit YAML

1. **Find the right file.** Bootique apps typically have YAMLs in one of:
   - `src/main/resources/` (built into the jar)
   - A repo-level `config/` or `etc/` directory (passed via `--config=`)
   - A `local.yml` or `dev.yml` overlay
   Ask if it's not obvious. The user may want the change in a specific overlay (e.g., dev-only) rather than the base config.
2. **Validate the path.** Check it against the discovered tree before writing. A typo or stale property name creates silent runtime failures (Bootique only complains at the level of the config root, not unknown leaves under it — Jackson-level `FAIL_ON_UNKNOWN_PROPERTIES` behavior is module-dependent).
3. **Preserve YAML structure.** Match indentation and key ordering used in the file. If a parent path doesn't exist yet, add the minimum nesting; don't reformat unrelated keys.
4. **Confirm units.** `Duration` accepts `30s`, `5m`, `1h`. Sizes accept `512kb`, `64mb`. Don't write a bare integer where the factory expects a `Duration`.

## Sub-flow C — Edit factory class

User wants to make a new setting configurable. Concretely:

1. Find the factory class for the relevant config root. (From discovery: each `ModuleCrate.config(prefix, X.class)` names the factory; nested factories are reached by following `@BQConfig` types.)
2. Add the setter:
   ```java
   @BQConfigProperty("Short human description used in --help-config")
   public void setRetryCount(int retryCount) {
       this.retryCount = retryCount;
   }
   ```
   - Field naming follows the existing factory's style (often `private` + setter; some factories use the builder-style `return this`).
   - Description goes in the annotation `value()`. Without it, `--help-config` shows the property without help text.
3. **If the property's type is itself a factory**, annotate that class with `@BQConfig("...")` so discovery recurses into it. Without `@BQConfig` on the type, the YAML key works at runtime but doesn't show up in `--help-config`.
4. **Hook the value into the runtime.** A setter is necessary but not sufficient — the factory has to actually use the field somewhere (typically inside its `create*()` method that the module's binder calls). Skip this only if the user is intentionally introducing the field for a follow-up commit.
5. Rebuild and re-run `--help-config` to confirm the new property surfaces.

## Sub-flow D — Override without editing

User wants to change a value without touching files. Options, in order of safety:

1. **Declared env var:** if the relevant module declares one via `BQCoreModule.extend(binder).declareVar("foo.bar", "FOO_BAR")`, point the user at the env var. Verify the declaration exists by grepping the module's `configure(Binder)` — don't assume `BQ_FOO_BAR` works.
2. **Layered config:** add a `--config=<overlay.yml>` with just the overrides.
3. **System property:** `-Dfoo.bar=value` on the JVM command line. Always works for any leaf. No declaration needed. Word of caution: this approach is prohibited for deployment. Can use this for quick tests only; never suggest this for production use.

## Anti-patterns to avoid

- **Don't invent properties.** If `--help-config` and the source don't show the property the user is asking about, it doesn't exist on this app — say so.
- **Don't edit unrelated config.** Make the smallest change that resolves the request.
- **Don't skip `@BQConfig` on a new factory type.** It's the line that makes the new node visible to `--help-config` and to this skill on its next run.
- **Don't suggest an env var without checking the binding.**
