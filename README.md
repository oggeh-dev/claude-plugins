# OGGEH — Claude Code plugin marketplace

This repository is a **Claude Code plugin marketplace** maintained by [OGGEH, Inc](https://oggeh.com). Marketplace name: `oggeh`.

A "marketplace" in Claude Code is just a catalog of plugins hosted on GitHub. Anyone can register this marketplace into their own Claude Code installation and install any plugin we ship from it.

## Quickstart

Inside Claude Code, run these two commands once to register the marketplace and install a plugin from it:

```
/plugin marketplace add oggeh-dev/claude-plugins
/plugin install cost-guard@oggeh
```

The first line tells your Claude Code installation about this marketplace (resolves the name `oggeh` to this GitHub repo). The second installs a specific plugin from it. Repeat the second command for each plugin you want.

## Plugins in this marketplace

Each plugin lives in its own dedicated repository. This catalog points at them via `github` sources, so `/plugin install <name>@oggeh` automatically clones the right repo.

| Plugin | Repository | Install command | What it does |
|---|---|---|---|
| `cost-guard` | [`oggeh-dev/claude-cost-guard`](https://github.com/oggeh-dev/claude-cost-guard) | `/plugin install cost-guard@oggeh` | Real-time USD cost enforcement for Claude Code. Halts runaway agentic loops mid-step before they burn through your budget. |

## Repository layout

This repo holds *only* the marketplace catalog and its metadata. Plugin source code lives in the plugins' own repos (linked above).

```
.
├── .claude-plugin/
│   └── marketplace.json    # marketplace catalog (lists every plugin)
├── LICENSE                 # MIT, applies to the catalog metadata
├── RELEASE.md              # release procedure for the catalog itself
└── README.md               # this file
```

For plugin-specific docs (configuration, halt mechanics, etc.) see each plugin's own README — e.g. [`oggeh-dev/claude-cost-guard`](https://github.com/oggeh-dev/claude-cost-guard).

## Releasing catalog updates

Maintainers: see [`RELEASE.md`](./RELEASE.md) for the procedure when a listed plugin ships a new version, when a new plugin is added, or when an existing plugin is removed.

## License

MIT — see [LICENSE](./LICENSE).
