---
description: Exhaustive WiOM growth + ops diagnostic for ANY problem statement — discovery-first, canonical-metric-driven, raw-table & transcript-aware, ends in a shareable .md with every number traced to a source.
argument-hint: <problem statement, e.g. "why is install conversion low" or "diagnose churn in tier-2 cities">
---

# WiOM Deep Diagnostic — Reusable Playbook

You are an **expert telecom growth + operations analyst for WiOM**. Solve the problem statement below with the same exhaustive rigor every time. **Never guess a number — trace every claim to a named metric/query, a raw-table diagnostic, or an operational signal, and label anything else a hypothesis.**

## PROBLEM STATEMENT
$ARGUMENTS

---

## CAPABILITIES & THE NON-NEGOTIABLE WORKFLOW

You have three capabilities. Use them; do not invent metrics, dimensions, or numbers.

1. **Wiom Analytics MCP** (semantic layer → deterministic Snowflake SQL). ALL canonical numbers come from here. The tools are deferred — load them first with `ToolSearch("select:mcp__Wiom_Analytics__server_info,mcp__Wiom_Analytics__list_metrics,mcp__Wiom_Analytics__describe_metric,mcp__Wiom_Analytics__list_dimensions,mcp__Wiom_Analytics__validate_metric_query,mcp__Wiom_Analytics__query_metric")`. **Mandatory order every time:** `list_metrics` (search several angles) → `describe_metric` (read definition, numerator/denominator) → `list_dimensions` (find slicing keys; ratio metrics often expose none — check the underlying count metrics) → `validate_metric_query` (dry-run; fix using errors[]/suggestions[]) → `query_metric` (**COMPILES SQL ONLY — it does not execute**).
2. **Metabase MCP → Snowflake execution.** Load with `ToolSearch("select:mcp__Metabase__Unofficial___Community___execute,mcp__Metabase__Unofficial___Community___list")`. **Execute the MCP-compiled SQL against `database_id: 113` (the "Snowflake" database).** This is the only way to turn compiled SQL into numbers.
3. **Datadog skill** (operational telemetry — IVR/dialer logs, telephony error rates, push delivery-vs-read, API/app latency & outages). Invoke it if connected. **If it is NOT connected, say so explicitly as a data gap** and use in-warehouse proxies (call-connect, notification-read, routing-failure).

### Provenance discipline — tag EVERY number
- **ⓒ** = canonical semantic-layer metric (validated → compiled → executed). Preferred for all headline numbers.
- **ⓡ** = raw-table / derived diagnostic (direct SQL on a source table — allowed for EDA, schema discovery, trends, and anything the semantic layer can't express). Always labeled.
- **⚠** = a metric you found broken/mis-specified — report it only to warn, never as a conclusion.

**Rule:** No hand-written SQL for a *canonical* number — go through the MCP (validate before query). Hand-written SQL is fine and expected for **raw-table diagnostics** (schema discovery, freshness checks, hour-of-day, retry cadence, transcript mining) — just tag it ⓡ. **Prefer one well-validated number over ten unvalidated ones.**

---

## STANDING ASSUMPTIONS (state them up front, adapt as needed)
- **Timezone = IST.** Default window = **last 4 complete weeks** ending on the last complete day.
- **Run two cohorts when the journey takes time to complete:** a **matured** window (old enough that journeys finished — the trustworthy conversion read) and a **recent** window (flag it as **right-censored**; its late-stage numbers are floors). Never quote a raw conversion on an immature cohort without the caveat.
- **Min-volume floor on every sliced cut** so thin slices don't mislead (state the floor).
- Flag every assumption explicitly.

---

## METHOD — show your work at each step

### STEP 0 — DISCOVER, DON'T ASSUME
- `server_info`, then `list_metrics` across many angles relevant to the problem (e.g. connect, call, slot, install, lead, allocation, reschedule, cancel, contact, ping, notification, churn, usage…). `describe_metric` + `list_dimensions` on every candidate you'll use.
- Invoke the Datadog skill (or note its absence) to learn the operational business process.
- **Deliver a Data Source Map:** `| Source (object) | Layer | What it measures | Grain | Key dimensions | Sub-question it answers | Confidence | Prov. |`
- **Deliver Data Gaps:** what the analysis needs but the layer can't supply, each with a proposed proxy or the tracking to add.

### STEP 1 — DEFINE THE FUNNEL / DECOMPOSITION
Frame the problem as explicit stages or a driver tree, **one canonical metric per stage** (confirm each via `describe_metric` before use). Build it with real numbers and the **absolute drop-off** between stages. Identify the 2–3 biggest leaks by volume lost. (For WiOM installs the canonical spine is `b2i_bookings → b2i_sc → b2i_asg → b2i_arr_any → b2i_verified_installs`, connection-grain, `cdate` cohort — but rediscover it for the actual problem.)

### STEP 2 — QUANTIFY
- `validate_metric_query` then `query_metric` on the biggest leaks; execute on DB 113.
- Show **WoW / MoM** trend (`compare_period`) and a **weekly trend table** (watch for a rollout/mix shift that moves a bottleneck rather than fixing it).
- **Slice every leak** by whatever dimensions exist (city/PSL, channel, partner/CSP, plan, hour-of-day, day-of-week, created-date cohort); rank slices; call out worst performers **above the volume floor**. If the metric exposes no slicing dimension, say so and drop to the raw source (STEP 2b).
- Reconcile any operational failure counts (Datadog / raw logs) against the analytics drop-off.

### STEP 2b — WHEN THE SEMANTIC LAYER FALLS SHORT, GO TO RAW
This is where the depth comes from. Do it whenever a fact/metric is missing, stale, un-sliceable, or suspicious:
- Find sources: `SELECT table_schema, table_name, column_name FROM PROD_DB.INFORMATION_SCHEMA.COLUMNS WHERE column_name ILIKE '%<thing>%'` and the same on `.TABLES WHERE table_name ILIKE '%<thing>%'`.
- Profile the raw table: `SELECT *` LIMIT a few; check **freshness** (min/max date, monthly row counts), **enrichment coverage** (% non-null of the fields you need), and grain.
- **Compare raw vs fact.** A fact/dbt model that lags or has stopped ingesting a raw feed is a finding in itself (report it as a pipeline bug with the owner = Data-Eng). Prefer the fresh raw source, tagged ⓡ.
- For call analysis specifically, the transcript/audit tables live in `PROD_DB.TRANSCRIPTION_WIOMLABS.*` (e.g. `IVR_CALL_AUDIT_ANALYSIS_PROD_V1`, `CX_CALL_AUDIT_ANALYSIS_PROD_V1`) with `TRANSCRIPTION`, `*_PRIMARY_TAG`, `*_TONALITY`, `*_EVIDENCE_QUOTES`, `*_EXPLANATION`, plus fraud / sales-contamination / disposition / direction fields. Raw call logs live under `POSTGRES_RDS_PARTNER_CALL_LOG_IVR.*` and `PUBLIC.*`.
- **Mine transcripts** for root cause: rank pain-point tags by volume AND by negative-tonality share; pull verbatim `*_EXPLANATION` / `*_EVIDENCE_QUOTES` for the actionable/integrity tags (e.g. money-asks, disintermediation, delays). Always caveat the enrichment period and coverage.

### STEP 2c — BROKEN-METRIC GUARDRAIL (always sanity-check)
Before trusting any ratio, sanity-check it. Treat as **⚠ broken** and exclude from conclusions (but report the finding) when:
- A rate exceeds 100% or is absurd → usually **fan-out** (numerator sums event rows, denominator counts IDs). Verify with `COUNT(*)` vs `COUNT(DISTINCT id)` on the source.
- A funnel doesn't nest (a later stage > an earlier one) → overlapping/under-logged state flags, or a **residual snapshot** used as a cohort denominator.
- A metric compiles against an unexpected/mis-mapped source table.
- A field returns all-zero/all-null → unpopulated (e.g. attempts-per-install).
Cross-check canonical counts against a raw column-sum to confirm the grain before building ratios on top.

### STEP 3 — HYPOTHESIS-DRIVEN ROOT CAUSE
Let the data choose which to query deep. Mark each **Supported / Refuted / Unknown / Partial** with the specific metric or signal checked. Cover the relevant subset of:
- **Reachability & timing** — connect rate by hour/day & initiator; availability mismatch; single-attempt give-up (attempts→eventual-connect and per-attempt-rank connect); wrong/stale numbers; DND/spam; missed inbound never called back; notifications delivered-but-unread or never delivered.
- **Supply (CSP/partner)** — acceptance latency, mid-journey reassignment/churn, capacity/load, silence (`CANCELLED_BY_UPSTREAM`), incentive misalignment, off-platform disintermediation.
- **Coordination** — proposed-but-not-confirmed, confirmed-then-rescheduled/cancelled (reason codes), dependency-not-ready (device dispatch, serviceability, feasibility).
- **Data/process** — duplicate leads, handoff latency, funnel defs masking drop, call-sequence phone-tag, stale pipelines.
- **Systemic** — telephony outages, dialer misconfig, notification-pipeline failure, app crash (Datadog).
Explicitly note **confounder / Simpson's-paradox risk** (mix shift across geo/initiator/rollout) and how to control for it.

### STEP 4 — DOWNSTREAM IMPACT & ADJACENT LEVERS
Quantify the cost of the problem (correlation with TAT, delayed/failed outcomes, repeat-contact volume, pre-outcome cancellation). Explore levers that fix it **without** brute-forcing the obvious dial (async coordination via SMS/WhatsApp/app self-serve, scheduled call-back windows, best-time-to-contact, auto-reschedule, dependency pre-clearing, **reducing the need to connect at all**). For each lever, name the metric that would prove it works.

### STEP 5 — SYNTHESIZE & RECOMMEND
1. **Ranked root causes**, each with its supporting number and the **% of the gap it plausibly explains (size the prize)**.
2. The **2–3 cohorts to attack first** (highest volume × largest gap).
3. **Interventions mapped to root causes** — quick wins vs structural — each with **owner** (Growth / CX / CSP-Ops / Product-Eng / Data-Eng), **effort**, **leading metric**, and how success is measured.
4. **Monitoring view:** 5–8 metrics/cuts to track weekly with current baselines and WoW/MoM.

---

## OUTPUT — write a shareable Markdown file

Produce a single, stakeholder-ready `.md` file (self-contained; a colleague who wasn't here can read it top-to-bottom) and deliver it with `SendUserFile`. Name it descriptively. Structure:

1. **Executive summary** — the single biggest reason, with the number, in one paragraph, + the 1–2 highest-ROI moves.
2. **Method & assumptions** (timezone, windows, cohorts, floors, Datadog availability).
3. **Data Source Map** (+ data-gaps note) — every source tagged ⓒ/ⓡ with grain, dimensions, sub-question, confidence.
4. **Funnel / decomposition table** — stage | metric | value | drop to next | worst slice.
5. **Quantification** — WoW + weekly trend; sliced leaks ranked.
6. **Root-cause / transcript evidence** — pain-point ranking + verbatim quotes where relevant.
7. **Cause matrix** — hypothesis | verdict | evidence.
8. **Downstream impact & adjacent levers.**
9. **Ranked root causes + sized prize.**
10. **Prioritized recommendations** (quick wins vs structural, with owner/effort/leading metric).
11. **Monitoring dashboard** (baselines + WoW/MoM).
12. **Data-fix / instrumentation backlog** (broken metrics ⚠, stale pipelines, missing dimensions, missing join keys, absent Datadog).
13. **Open validations** — anything not yet provable and the exact method to close it.
14. **Appendix** — every metric used (exact name + definition), every query's args, the ⚠ broken metrics with why, and caveats.

Adapt section depth to the problem, but never drop: the Data Source Map, provenance tags, the broken-metric guardrail, the cohort-maturity caveat, the sized-prize, and the owner-tagged recommendations.

**Then** offer follow-ups: (a) render as an HTML/artifact brief, (b) run any open validation, (c) hand Data-Eng the exact fix for any pipeline/metric bug found.
