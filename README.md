# oggeh — Claude Code plugin marketplace

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

| Plugin | Install command | What it does |
|---|---|---|
| [`cost-guard`](./plugins/cost-guard/) | `/plugin install cost-guard@oggeh` | Real-time USD cost enforcement for Claude Code. Halts runaway agentic loops mid-step before they burn through your budget. |

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json    # marketplace catalog
├── plugins/
│   └── cost-guard/         # the cost-guard plugin (its own .claude-plugin/plugin.json)
│       ├── README.md       # plugin documentation
│       ├── TESTING.md      # local-test procedures
│       ├── CHANGELOG.md
│       ├── bin/, hooks/, skills/, pricing.json
│       └── .claude-plugin/plugin.json
├── LICENSE                 # MIT, applies to the whole repo
├── RELEASE.md              # release procedure for maintainers
└── README.md               # this file
```

For plugin-specific docs (configuration, halt mechanics, indicator setup, etc.) see [`plugins/cost-guard/README.md`](./plugins/cost-guard/README.md).

## Releasing a new version

Maintainers: see [`RELEASE.md`](./RELEASE.md) for the step-by-step release procedure (versioning policy, manifest bumps, tagging, and the pre-flight checklist).

## License

MIT — see [LICENSE](./LICENSE).

 Copyright (c) 2026 OGGEH, Inc.
 