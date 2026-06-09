# bootique-ai-plugin

A Claude Code plugin (and marketplace) for [Bootique](https://bootique.io) — the
modular, DI-based runtime for runnable Java apps — and its standard modules.

## Layout

```
.claude-plugin/marketplace.json   # marketplace descriptor
plugin/                           # the "bootique" plugin (skills + references)
```

## Install

```
# register this repo as a marketplace
/plugin marketplace add bootique/bootique-ai-plugin

# install the plugin
/plugin install bootique@bootique
```

See [`plugin/README.md`](plugin/README.md) for what the plugin does and the
skills it ships.
