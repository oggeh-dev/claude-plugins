# Catalog release process

This repo IS the `oggeh` marketplace catalog — it lists plugins by reference, but it does not host any plugin code. Each plugin lives in its own repository (e.g. `cost-guard` lives at [`oggeh-dev/claude-cost-guard`](https://github.com/oggeh-dev/claude-cost-guard)) and follows its own release flow.

This catalog only needs an update when one of the following happens:

1. **A listed plugin ships a new version** — bump that plugin's `version` field in `marketplace.json` to match the plugin repo's new `plugin.json` version.
2. **A new plugin is added to the catalog** — add a new entry to the `plugins[]` array.
3. **An existing plugin is removed from the catalog** — remove its entry from `plugins[]`.
4. **A plugin's source URL or other catalog metadata changes** — edit the relevant entry.

The catalog is **not tagged per plugin release**. Tags belong with the plugins, not the catalog. The catalog's own `metadata.version` field tracks structural changes to the catalog itself (a plugin was added/removed, source types changed) — bump it when those things happen, leave it alone otherwise.

---

## When a listed plugin ships a new version (most common)

Triggered by: a plugin's repo (e.g. `oggeh-dev/claude-cost-guard`) just released `cost-guard--vX.Y.Z`.

```bash
# 1. Edit .claude-plugin/marketplace.json — bump plugins[N].version to match.
#    For cost-guard, find: { "name": "cost-guard", "version": "..." }
#    and update version to the new value.

# 2. Validate.
claude plugin validate .

# 3. Commit + push.
git add .claude-plugin/marketplace.json
git commit -m "chore(catalog): cost-guard X.Y.Z"
git push origin main
```

That's it. No catalog-side tag. Users on auto-update will pick up the new plugin version on next session start; users on manual update can run `/plugin update cost-guard@oggeh`.

---

## When adding a new plugin

```bash
# 1. Edit .claude-plugin/marketplace.json — append to plugins[]:
#    {
#      "name": "<plugin-name>",
#      "source": { "source": "github", "repo": "oggeh-dev/<plugin-repo>" },
#      "description": "...",
#      "version": "<plugin-version>",
#      "author": { "name": "OGGEH, Inc" },
#      "license": "MIT",
#      ...
#    }

# 2. Bump metadata.version (catalog has more plugins than before).

# 3. Update README.md — add a row to the "Plugins in this marketplace" table.

# 4. Validate.
claude plugin validate .

# 5. Commit + push.
git add .claude-plugin/marketplace.json README.md
git commit -m "feat(catalog): list <plugin-name>"
git push origin main
```

---

## When removing a plugin

```bash
# 1. Remove its entry from plugins[] in marketplace.json.
# 2. Bump metadata.version.
# 3. Update README.md — remove its row.
# 4. Validate.
# 5. Commit:
git commit -am "chore(catalog): remove <plugin-name>"
# 6. Push.
git push origin main
```

Users with the plugin already installed keep working until they uninstall manually — Claude Code does not auto-uninstall when a marketplace de-lists a plugin.

---

## Catalog `metadata.version` policy

| Bump | When |
|---|---|
| MAJOR | The catalog name changes, or a structural breaking change to the schema. |
| MINOR | A plugin is added or removed; a plugin's `source` type changes; the catalog's identity (`name`, `owner`) changes. |
| PATCH | Description tweak, keyword adjustment, README polish. Per-plugin `version` bumps **do not** require a catalog `metadata.version` bump. |

---

## Pre-flight checklist

- [ ] `claude plugin validate .` passes.
- [ ] All `plugins[N].version` fields match the corresponding plugin repos' latest plugin.json versions.
- [ ] All `plugins[N].source` entries point at repos that exist and contain a valid `plugin.json` at root.
- [ ] README.md plugin table reflects the current catalog (no stale entries, no missing ones).
- [ ] `git status` is clean.

---

## Reference

- [Claude Code plugin docs](https://code.claude.com/docs/en/plugins)
- [Marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces)
- [Semantic Versioning](https://semver.org/spec/v2.0.0.html) (for the catalog's own `metadata.version`)
- Per-plugin release flow: see each plugin repo's own `RELEASE.md` (e.g. [`oggeh-dev/claude-cost-guard/RELEASE.md`](https://github.com/oggeh-dev/claude-cost-guard/blob/main/RELEASE.md)).
