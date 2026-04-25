# oggeh — Claude Code plugins

This repository is a **Claude Code plugin marketplace** maintained by [OGGEH, Inc](https://github.com/oggeh-dev). Marketplace name: `oggeh`.

## Plugins in this marketplace

| Plugin | Install command | What it does |
|---|---|---|
| [`cost-guard`](./plugins/cost-guard/) | `/plugin install cost-guard@oggeh` | Real-time USD cost enforcement for Claude Code. Halts runaway agentic loops mid-step before they burn through your budget. |

## Add this marketplace

Inside Claude Code:

```
/plugin marketplace add oggeh-dev/claude-cost-guard
```

Then install any plugin from the table above with `/plugin install <name>@oggeh`.

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
└── README.md               # this file
```

For plugin-specific docs (configuration, halt mechanics, indicator setup, etc.) see [`plugins/cost-guard/README.md`](./plugins/cost-guard/README.md).

## Releasing a new version

Maintainers: see [`RELEASE.md`](./RELEASE.md) for the step-by-step release procedure (versioning policy, manifest bumps, tagging, and the pre-flight checklist).

## License

MIT — see [LICENSE](./LICENSE).
