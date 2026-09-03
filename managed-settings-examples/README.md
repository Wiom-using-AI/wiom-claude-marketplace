# Per-function managed-settings examples

IT/admin deploys one of these as `managed-settings.json` to each function's device group (via MDM / GPO / Intune / Jamf). These already point at `Wiom-using-AI/wiom-claude-marketplace`.

- `all-functions-core.json` — the minimum everyone gets (core diagnostic).
- `growth.json`, `cx.json`, `csp-ops.json` — core **plus** that function's plugin.

`enabledPlugins` auto-installs and enables the listed plugins. Add the same marketplace block to every file so the group knows where to fetch from.
