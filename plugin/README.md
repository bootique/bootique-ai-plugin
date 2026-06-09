# bootique — Claude Code plugin

Bootique workflows for Claude Code.

This plugin teaches Claude how to build and extend [Bootique](https://bootique.io)
applications — bootstrapping a runnable app, authoring modules, wiring services
through DI, defining configuration, and writing commands.

The plugin targets downstream **users of Bootique** building their own Java apps,
as well as authors of reusable Bootique modules.

## Install

The plugin is distributed from the Bootique GitHub repository:
**https://github.com/bootique/bootique-ai-plugin**. Inside Claude Code:

```
# register the "marketplace"
/plugin marketplace add bootique/bootique-ai-plugin

# install the plugin
/plugin install bootique@bootique
```

## What's in here

```
plugin/
├── .claude-plugin/plugin.json        # manifest
├── README.md                         # this file
├── skills/                           # auto-triggering workflows
│   ├── bootique-config/              # discover & edit the config surface
│   └── bootique-run/                 # build and run a Bootique app
└── references/                       # source-of-truth docs loaded by skills
    ├── bootique-overview.md          # module / config / CLI fundamentals (4.0)
    ├── runnable-jar-discovery.md     # locate the runnable jar in Maven
    └── config-discovery.md           # extract the effective config tree
```

Each skill is a thin trigger that loads the right reference docs and walks Claude
through the workflow.

### Skills

- **bootique-config** — discover the configuration tree of a Bootique app and
  apply changes (YAML files, environment variables, or `@BQConfig` factory
  classes).
- **bootique-run** — run a Bootique app: start a server, execute a single job,
  or invoke `--help` / `--help-config` for inspection.

### Conventions assumed

- Build system: **Maven** (single- or multi-module).
- Runnable jar produced by either the *"Runnable Jar with `lib` Folder"* or
  *"Runnable Jar with Dependencies"* recipe — both expose `java -jar <path>` as
  the entry point.
- Bootique version: **4.x** (`BQModule.crate()` returning a `ModuleCrate`,
  `@BQConfig` / `@BQConfigProperty` annotations). Pre-4.0 APIs are intentionally
  out of scope.
