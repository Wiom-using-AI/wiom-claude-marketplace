# WiOM Claude Code Marketplace

Internal Claude Code tools for WiOM. **One marketplace, many plugins, enabled per function.** The core plugin ships **`/wiom-diagnose`** — an exhaustive, repeatable growth + ops diagnostic for any problem statement.

---

## Quick start (any user)

```
/plugin marketplace add Wiom-using-AI/wiom-claude-marketplace
```
```
/plugin install wiom-analytics@wiom
```

Then run the diagnostic with any problem statement:

```
/wiom-diagnose why is install conversion low
```

> Requires the data MCP servers to be connected (see **Prerequisites**). Without them the command runs the method but can't return numbers.

---

## Using `/wiom-diagnose`

Type the command followed by a plain-English problem statement. Examples:

```
/wiom-diagnose why are timely connects between customers and CSPs low
/wiom-diagnose diagnose pre-install cancellation in tier-2 cities
/wiom-diagnose where is the booking-to-install funnel leaking this month
/wiom-diagnose why is churn rising for R15 customers
```

**What you get back:** a stakeholder-ready `.md` report that
- opens with the single biggest reason + the number,
- maps every data source (tagged **ⓒ** canonical / **ⓡ** raw / **⚠** broken),
- builds the funnel with absolute drop-offs, WoW + weekly trend, and sliced leaks,
- mines call transcripts for root-cause pain points,
- ranks root causes with a **sized prize**, then gives owner-tagged recommendations, a monitoring view, and a data-fix backlog.

It always runs canonical numbers through the Wiom Analytics semantic layer → Snowflake, drops to raw tables when the layer falls short, and flags any broken metric instead of trusting it.

---

## Prerequisites (per user)

`/wiom-diagnose` reads live data, so each user needs these connected in their Claude Code:

| Dependency | Purpose |
|---|---|
| **Wiom Analytics MCP** | The canonical semantic layer (metrics → deterministic SQL) |
| **Metabase MCP** (access to `database_id: 113`, Snowflake `PROD_DB`) | Executes the compiled SQL |
| **Datadog skill** *(optional)* | Operational telemetry; the command flags its absence as a data gap |

These are **not bundled** in the plugin — connect them via your normal MCP configuration or org managed settings.

---

## Admin rollout — enable per function (recommended for "everyone")

Instead of each person installing, IT deploys a `managed-settings.json` to each function's device group. The listed plugins auto-install; **core is enabled for all, function plugins are scoped per group.**

Ready-made files in [`managed-settings-examples/`](managed-settings-examples/) (already pointed at this repo):

| File | Enables |
|---|---|
| `all-functions-core.json` | `wiom-analytics` (core) — the baseline everyone gets |
| `growth.json` | core + `wiom-growth` |
| `cx.json` | core + `wiom-cx` |
| `csp-ops.json` | core + `wiom-csp-ops` |

Deploy the matching file (renamed to `managed-settings.json`) to each group via MDM/GPO/Intune/Jamf:

- **Windows:** `C:\ProgramData\ClaudeCode\managed-settings.json`
- **macOS:** `/Library/Application Support/ClaudeCode/managed-settings.json`
- **Linux:** `/etc/claude-code/managed-settings.json`

---

## Managing the marketplace (users)

```
/plugin marketplace update wiom     # pull the latest plugin versions
/plugin uninstall wiom-analytics@wiom
/plugin marketplace remove wiom
```

Use `/plugin` on its own to browse installed plugins and what each provides.

---

## Repository layout

```
wiom-claude-marketplace/
├─ .claude-plugin/marketplace.json     # lists every plugin
├─ plugins/
│   └─ wiom-analytics/                 # CORE — /wiom-diagnose (enable for ALL functions)
│       ├─ .claude-plugin/plugin.json
│       ├─ commands/wiom-diagnose.md
│       └─ README.md
│   # add function plugins here: wiom-growth/, wiom-cx/, wiom-csp-ops/ ...
└─ managed-settings-examples/          # per-function auto-enable snippets for IT
```

---

## Adding a new function plugin

1. Create `plugins/wiom-<function>/` with `.claude-plugin/plugin.json` and a `commands/` (and/or `agents/`, `skills/`) folder.
2. Add an entry for it in `.claude-plugin/marketplace.json`.
3. Add `wiom-<function>@wiom` to that function's `enabledPlugins` in `managed-settings-examples/`.
4. Commit and push — users pick it up with `/plugin marketplace update wiom`.

Keep shared logic in `wiom-analytics`; put only function-specific commands in the function plugin.

---

## Publishing changes

After editing anything in this repo:

```
git add . && git commit -m "describe change" && git push
```

Users then run `/plugin marketplace update wiom` to get it.

---

## Support

Questions or new-command requests: **analytics@wiom.in**.
