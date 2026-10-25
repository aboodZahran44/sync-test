# Sync test plugin

A tiny plugin used to answer one question: **when we push a change to GitHub, does the plugin in Claude update by itself?**

It contains one skill and one agent. Each one replies with a version marker, so you can tell at a glance which version Claude has loaded.

| | v1 (now) | v2 (after the update) |
|---|---|---|
| Skill `sync-check` | `Version: v1` / `Color: GREEN` | `Version: v2` / `Color: BLUE` |
| Agent `sync-tester` | `Version: v1` / `Animal: CAMEL` | `Version: v2` / `Animal: FALCON` |
| `plugin.json` version | `1.0.0` | `1.0.1` |

## Layout

```
.claude-plugin/marketplace.json          ← the "store" file: lists the plugins in this repo
plugins/sync-test/
  .claude-plugin/plugin.json             ← the plugin's name and version
  skills/sync-check/SKILL.md             ← the test skill
  agents/sync-tester.md                  ← the test agent
```

The plugin sits in a subfolder on purpose, like `marketing_vertical/oman-marketing-agents` in the real repo, so the test matches the real setup.

Both manifests pass `claude plugin validate`.

## What a GitHub update needs (checked with Claude Code 2.1.310)

1. **Bump `version` in `plugin.json` on every change.** Without a bump, Claude Code reports "already at the latest version" and keeps the old files.
2. **Something has to pull the update.** A push alone changes nothing. Either auto-update is on for the marketplace, or someone runs the update commands.
3. **Restart Claude Code.** An update is applied on the next start.

## Test steps (Claude Code)

### 1. Push v1 to GitHub
```bash
git init
git add .
git commit -m "Sync test plugin v1"
git branch -M main
git remote add origin https://github.com/<your-username>/sync-test.git
git push -u origin main
```

### 2. Add the marketplace and install
In PowerShell:
```
claude plugin marketplace add <your-username>/sync-test
claude plugin install sync-test@abwaab-sync-test
claude plugin list
```
`claude plugin list` should show `sync-test@abwaab-sync-test`, Version `1.0.0`.

### 3. Turn on auto-update
Run `claude`, type `/plugin`, go to **Marketplaces**, select `abwaab-sync-test`, choose **Enable auto-update**. Then type `/exit`.

### 4. Check v1
Run `claude` and type:
- `/sync-test:sync-check` → `Version: v1` / `Color: GREEN`
- `Use the sync-tester agent` → `Version: v1` / `Animal: CAMEL`

### 5. Push v2
- `plugins/sync-test/skills/sync-check/SKILL.md`: `v1` → `v2`, `GREEN` → `BLUE`
- `plugins/sync-test/agents/sync-tester.md`: `v1` → `v2`, `CAMEL` → `FALCON`
- `plugins/sync-test/.claude-plugin/plugin.json`: `"1.0.0"` → `"1.0.1"`

```bash
git add .
git commit -m "Sync test plugin v2"
git push
```

### 6. The real test: no update commands
Start `claude`, wait a minute, `/exit`, start `claude` again, then run the two checks.
- **v2 / BLUE / FALCON** → auto-update works.
- **Still v1** → run `claude plugin list` and check the version. If it is still `1.0.0`, auto-update didn't run.

### 7. Manual fallback (only if step 6 failed)
```
claude plugin marketplace update abwaab-sync-test
claude plugin update sync-test@abwaab-sync-test
```
Then restart `claude` and check again.

### 8. Optional: confirm the version bump is required
Change `BLUE` → `RED` without changing the version, push, and restart. You should still see `BLUE`.

### Cleanup
```
claude plugin uninstall sync-test@abwaab-sync-test
claude plugin marketplace remove abwaab-sync-test
```
