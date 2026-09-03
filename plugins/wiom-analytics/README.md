# wiom-analytics

Core WiOM analytics plugin. Ships the **`/wiom-diagnose`** command — an exhaustive, repeatable growth + ops diagnostic for any problem statement.

## What it does
Give it a problem statement and it runs the full playbook:
- Discovery-first (Data Source Map + Data Gaps)
- Canonical numbers via the Wiom Analytics semantic layer → executed on Snowflake (Metabase DB 113)
- Raw-table fallback + transcript mining when the semantic layer falls short
- Broken-metric guardrail, provenance tags (ⓒ canonical / ⓡ raw / ⚠ broken)
- Matured vs. right-censored cohorts, min-volume floors, IST
- Ends in a stakeholder-ready `.md` with sized prize, owner-tagged recommendations, and a monitoring view

## Usage
```
/wiom-diagnose why is install conversion low
/wiom-diagnose diagnose pre-install cancellation in tier-2 cities
```

## Prerequisites (per user)
The command relies on tools that must be connected in each user's Claude Code:
- **Wiom Analytics MCP** (the semantic layer)
- **Metabase MCP** with access to `database_id: 113` (Snowflake `PROD_DB`)
- Optionally the **Datadog** skill for operational telemetry (the command flags its absence as a gap)

Add these via your org's MCP configuration (see the top-level marketplace README).
