# Release process

How to ship a new release of any plugin in this marketplace. Today this only covers `cost-guard`, but the same flow applies if more plugins land here later.

This repo is **two manifests sharing one git history**:

- `plugins/cost-guard/.claude-plugin/plugin.json` — the plugin's own version. Claude Code uses this field to decide whether a user has the latest.
- `.claude-plugin/marketplace.json` — the catalog. Each `plugins[]` entry mirrors the plugin's version; the catalog itself has its own optional `metadata.version`.

The `claude plugin tag` command refuses to create a release tag unless the plugin manifest and the matching marketplace entry agree on the version, which means **you cannot accidentally ship one without the other**.

---

## Versioning policy

We follow [semantic versioning](https://semver.org/) — `MAJOR.MINOR.PATCH`.

| Bump | When | Examples for cost-guard |
|---|---|---|
| **PATCH** (`0.1.0 → 0.1.1`) | Bug fix, doc update, pricing-table refresh — anything that doesn't change behavior users rely on. | Halt-banner typo. JSONL parser bug. Anthropic pricing changed. README clarification. New default already shipped via env-var override. |
| **MINOR** (`0.1.0 → 0.2.0`) | Backwards-compatible feature: anything that adds capability without changing or removing existing behavior. | New `/cost-guard:*` skill. New `userConfig` field (with a default that preserves prior behavior). New halt event registered. New indicator metric. |
| **MAJOR** (`0.1.0 → 1.0.0`) | Breaking change: anything that requires users to update their config, mental model, or expectations. | Renaming a `userConfig` key. Removing or renaming a skill. Changing the meaning of an existing field. Moving `${CLAUDE_PLUGIN_DATA}` files. Switching halt semantics. |

**Pre-1.0 exception:** while the leading version is `0.x.y`, breaking changes are allowed to bump `MINOR` (`0.1.0 → 0.2.0`) instead of `MAJOR`. The leading `0` signals the API is still stabilizing. Once we ship `1.0.0`, strict semver applies and breaking changes go to `MAJOR`.

---

## Three files to update on every release

### 1. `plugins/cost-guard/.claude-plugin/plugin.json`

Bump the `version` field:

```json
{
  "name": "cost-guard",
  "version": "0.2.0",   // <- bump this
  ...
}
```

This is the field Claude Code uses to detect "has the user got the latest?". If you don't bump it, users running `/plugin update cost-guard@oggeh` will see "already at the latest version" and miss your release. **Skipping this bump is the most common release mistake.**

### 2. `.claude-plugin/marketplace.json`

Two version fields live here:

```json
{
  "metadata": {
    "version": "0.1.0"          // catalog version — usually unchanged per release
  },
  "plugins": [
    {
      "name": "cost-guard",
      "version": "0.2.0",       // <- bump this to match plugin.json
      ...
    }
  ]
}
```

- **`plugins[0].version`** — must match `plugin.json`'s version exactly. The `claude plugin tag` command verifies this.
- **`metadata.version`** — the marketplace catalog's version. Bump only when *the catalog itself* changes (a plugin was added or removed, `source` paths changed, owner info changed). A new release of an existing plugin does *not* require bumping `metadata.version`.

### 3. `plugins/cost-guard/CHANGELOG.md`

Add a new section at the top, above the previous release entry:

```markdown
## [0.2.0] — 2026-MM-DD

### Added
- New skill `/cost-guard:foo` …

### Changed
- Default `rate_per_min_usd` raised from 0.50 to 0.75 …

### Fixed
- Halt banner displayed `step` cost as `$NaN` when …

### Removed
- Deprecated `/cost-guard:set-bar` skill (was renamed in 0.1.5) …
```

Use `### Added`, `### Changed`, `### Fixed`, `### Deprecated`, `### Removed`, and `### Security` headings as needed (these are the [Keep a Changelog](https://keepachangelog.com/) conventions). The CHANGELOG is what reviewers and users read first — make it complete and human-readable.

---

## Step-by-step release procedure

Assuming you're starting from a clean `main` branch with the work to be released already merged:

```bash
# 1. (Optional) Branch for the release commit. You can also do this on main.
git checkout -b release/v0.2.0

# 2. Edit the three files: plugin.json, marketplace.json, CHANGELOG.md.

# 3. Sanity-check that both manifests still validate.
claude plugin validate .                           # marketplace
claude plugin validate plugins/cost-guard          # plugin

# 4. Confirm the versions match (the tag command will fail otherwise).
jq -r '.version' plugins/cost-guard/.claude-plugin/plugin.json
jq -r '.plugins[0].version' .claude-plugin/marketplace.json
# These two must print the same string.

# 5. Commit. A single release commit is fine.
git add plugins/cost-guard/.claude-plugin/plugin.json \
        plugins/cost-guard/CHANGELOG.md \
        .claude-plugin/marketplace.json
git commit -m "release: cost-guard v0.2.0"

# 6. Merge to main (skip if step 1 was on main directly).
git checkout main
git merge --ff-only release/v0.2.0
git branch -d release/v0.2.0

# 7. Dry-run the tag. This validates plugin.json + marketplace.json agree
#    and prints the tag name without creating it.
claude plugin tag plugins/cost-guard --dry-run

# Expected output:
#   Would create tag: cost-guard--v0.2.0

# 8. Create the tag and push it (one step). Annotated tags carry a message.
claude plugin tag plugins/cost-guard --push -m "cost-guard v0.2.0"

# 9. Push the release commit.
git push origin main
```

**Tag format** — the command always produces `{plugin-name}--v{version}` (e.g., `cost-guard--v0.2.0`). The double dash is intentional and reserved for plugin tags by Claude Code.

---

## How users receive the update

After your push:

```
/plugin marketplace update oggeh
/plugin update cost-guard@oggeh
```

If a user has auto-update enabled for the `oggeh` marketplace (`/plugin` → Marketplaces tab, toggle on the marketplace row), Claude Code pulls the new commit on next start and prompts them to run `/reload-plugins`.

**Auto-update default for third-party marketplaces is OFF** — Anthropic's official marketplace ships it ON. Users who add `oggeh-dev/claude-cost-guard` get a manual-update marketplace by default; they can flip the toggle if they want auto-updates.

---

## Special-case releases

### Pricing-table refresh

When Anthropic publishes new model prices, edit `plugins/cost-guard/pricing.json` and bump the patch version (e.g., `0.2.0 → 0.2.1`). The `_metadata.last_verified` field inside `pricing.json` should be updated to today's date so users / reviewers can see when the table was last checked.

CHANGELOG entry under `### Changed`:
```markdown
- Pricing table refreshed against Anthropic's published rates as of YYYY-MM-DD.
```

### Hot fix on a published release

If a release is broken in production (a halt is firing wrongly, the indicator crashes, etc.):

1. Branch from the broken release's tag: `git checkout -b hotfix/v0.2.1 cost-guard--v0.2.0`.
2. Fix the bug. Add a CHANGELOG entry under `### Fixed`.
3. Bump patch in plugin.json + marketplace.json.
4. Run the standard release procedure (steps 3–9 above).

### Adding a new plugin to the marketplace

When a *second* plugin lands in this repo (e.g., `oggeh-data-tracker`):

1. Add `plugins/<new-plugin>/` with its own `.claude-plugin/plugin.json`, `bin/`, etc.
2. Add the new entry to `marketplace.json`'s `plugins` array.
3. **Bump `metadata.version`** in marketplace.json — this is the case it exists for.
4. CHANGELOG entry at the marketplace level (a `RELEASE_NOTES.md` could be added later if there's enough catalog churn to need one).
5. Tag *each plugin* whose version moved using `claude plugin tag plugins/<name>`.

---

## Pre-flight checklist

Before tagging:

- [ ] `claude plugin validate .` passes.
- [ ] `claude plugin validate plugins/cost-guard` passes.
- [ ] `plugin.json.version` and `marketplace.json.plugins[0].version` are identical.
- [ ] CHANGELOG has a dated section for the new version with all user-visible changes documented.
- [ ] If pricing.json changed, `_metadata.last_verified` is today.
- [ ] All hook commands still resolve (`grep CLAUDE_PLUGIN_ROOT plugins/cost-guard/hooks/hooks.json`).
- [ ] `git status` is clean — no stray local files.
- [ ] `claude plugin tag plugins/cost-guard --dry-run` prints the expected tag name.

After pushing:

- [ ] The commit appears on GitHub at `oggeh-dev/claude-cost-guard`.
- [ ] The tag `cost-guard--v<NEW>` appears in the GitHub releases / tags view.
- [ ] In a fresh Claude Code session: `/plugin marketplace update oggeh` then `/plugin update cost-guard@oggeh` pulls the new version.
- [ ] `/cost-guard:status` shows `enabled: true` and the new behavior is in effect.

---

## Reference

- Claude Code plugin docs: <https://code.claude.com/docs/en/plugins>
- Marketplace docs: <https://code.claude.com/docs/en/plugin-marketplaces>
- `claude plugin tag --help`: validates plugin/marketplace agreement and creates `{name}--v{version}` tags.
- Keep a Changelog: <https://keepachangelog.com/en/1.1.0/>
- Semantic Versioning: <https://semver.org/spec/v2.0.0.html>
