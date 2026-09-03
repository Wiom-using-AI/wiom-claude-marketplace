# WiOM Claude Code Marketplace

Single source of truth for WiOM's internal Claude Code tools. **One marketplace, many plugins, enabled per function.**

```
wiom-claude-marketplace/
├─ .claude-plugin/marketplace.json     # lists every plugin
├─ plugins/
│   └─ wiom-analytics/                 # CORE — /wiom-diagnose (enable for ALL functions)
│       ├─ .claude-plugin/plugin.json
│       ├─ commands/wiom-diagnose.md
│       └─ README.md
│   # add function plugins here: wiom-growth/, wiom-cx/, wiom-csp-ops/, wiom-data/ ...
└─ managed-settings-examples/          # per-function auto-enable snippets for IT
```

## One-time setup (repo owner)
1. Push this folder to a repo under your org, e.g. `your-org/wiom-claude-marketplace`.
2. Replace `your-org/wiom-claude-marketplace` everywhere in `managed-settings-examples/` with the real repo.
3. Ensure users have the required MCP servers connected (Wiom Analytics + Metabase DB 113; optional Datadog).

## How each function gets it

### Option 1 — Self-serve (any user)
```
/plugin marketplace add your-org/wiom-claude-marketplace
/plugin install wiom-analytics@wiom
```

### Option 2 — Admin-enforced per function (recommended for "everyone")
IT deploys a `managed-settings.json` to each function's device group. Everyone in that group gets the listed plugins auto-installed, no user action. Core is enabled for all; function plugins are scoped per group. See `managed-settings-examples/`.

Managed-settings location on each machine:
- **Windows:** `C:\ProgramData\ClaudeCode\managed-settings.json`
- **macOS:** `/Library/Application Support/ClaudeCode/managed-settings.json`
- **Linux:** `/etc/claude-code/managed-settings.json`

## Adding a new function plugin
1. Create `plugins/wiom-<function>/` with `.claude-plugin/plugin.json` and a `commands/` (and/or `agents/`, `skills/`) folder.
2. Add an entry to `.claude-plugin/marketplace.json`.
3. Add `wiom-<function>@wiom` to that function's managed-settings `enabledPlugins`.

Function plugins can reuse the core: keep shared logic in `wiom-analytics`, put only function-specific commands in the function plugin.
