---
name: debugging
description: Investigates bugs and root causes using Grafana (Loki, read-only) and Metabase (read-only). Use when the user asks to debug, investigate, search logs, look in prd/uat, check production data, find the cause, find the root cause, RCA, diagnose, or explain why something is failing. Must use Grafana MCP for logs and Metabase MCP for DB evidence; must never perform destructive MCP operations via MCP; report findings using the output template. Load the matching system config from systems/ before querying.
metadata:
  mcp-servers: grafana, metabase
  version: 1.0.0
---

# Debugging

Use this skill when investigating production or UAT behaviour in a Letterfest system. Pair with **systematic-debugging** (if available in the repo) for the general debug process; this skill covers **where and how** to pull logs and database evidence.

## Before you query

1. Identify the **system** being investigated (repo name or user-stated target).
2. Read the system config at `systems/<system>.md` (e.g. [systems/octopus.md](systems/octopus.md)).
3. Open the **repositories** listed in that config for code context (primary repo first; related repos when the issue crosses systems).
4. If no config exists, read [systems/_template.md](systems/_template.md), ask the user for the missing details, or stop and tell them to add a system config.
5. For MCP setup: read [setup-metabase-mcp](../../../metabase/skills/setup-metabase-mcp/SKILL.md) and [grafana-cloud-mcp-tools](../../../grafana-cloud/skills/grafana-cloud-mcp-tools/SKILL.md) when tools are unavailable or need auth.

## Read-only rules

### MCP (always)

Grafana and Metabase MCP usage is **read-only only**, including when the user later asks for a fix.

**Allowed:** `query_loki_logs`, `query_loki_stats`, `list_datasources`, `query_prometheus`, Metabase `search`, `query`, `execute_query` (SELECT-only on allowed databases in the system config).

**Forbidden:** Any write/update/delete via MCP — dashboards, alerts, annotations, incidents, migrations, or mutating Metabase/Grafana queries (`UPDATE`, `INSERT`, `DELETE`). You may **recommend** operational fixes (SQL, manual steps) for the user to run outside MCP; never execute them via MCP.

### Code and remediation (by user intent)

| User says | Action |
| --- | --- |
| debug / investigate / find the cause / root cause / RCA / why is X / what's going on | **Report using the output template.** Include **Immediate remediation** when a safe, case-specific operational fix is known; otherwise state none identified. No code edits. |
| propose a fix / suggest a fix / how would you fix | **Written plan using the output template** — immediate remediation (if any) **and** code fix. Switch to plan mode if in agent mode. **Do not implement.** |
| fix X / debug and fix / find the cause and fix | Debug first (this skill), then implement the **code fix**. Still document immediate remediation in the report for the user to run if needed. |

Default deliverable: completed output template. When unsure which bucket applies, ask before editing code.

## Logs — Grafana MCP

### Datasource

Loki datasource name: **`grafanacloud-letterfest-logs`**. Resolve UID via `list_datasources` if needed.

### Tools

- Primary: `query_loki_logs` (read tool schema first; pass `datasourceUid` and `logql`).
- Wide time ranges: run `query_loki_stats` first to avoid huge result sets.

### Query construction

Use the **stream selector**, **log format**, **common fields**, and **example queries** from the system config. General patterns:

- Start with the system config's stream selector for `<env>` and `<component>`.
- Pipe `| logfmt` when logs use logfmt (`key=value`); prefer logfmt field filters over `|~` when the field is known.
- Filter on identifiers from the system config (`level="error"`, order refs, job ids, etc.).
- **Errors / stack traces:** Errors may include `stacktrace`, but Loki often emits **one line per stack frame**. Start from the first `level="error"` line with known identifiers; widen the time window or `|~` on class names / stack fragments if the full trace is split.

## Data — Metabase MCP

### Tools

- `search` — find tables/metrics (read schema first).
- `query` or `execute_query` — read data; **SELECT only**.

### Database selection

Use the **Metabase databases** table in the system config. Rules that apply to every system:

- Match Metabase to **both** the service and Loki `<env>`. Default `<env>` is `prd`.
- **Never query PRD databases for a UAT investigation** (and vice versa).
- Query only schemas and tables listed in the system config (usually `public`).
- **`search` may return the same table name on multiple connections** — use the row for the correct env from the system config, not a different env or **Data Warehouse**.

### Banned databases

**Never query `Data Warehouse`** in Metabase. If `search` returns a table on Data Warehouse (e.g. `database_schema` such as `octo_backoffice`, `octo_fulfilment`, `shopify`), use the matching operational `PRD | …` or `UAT | …` database from the system config for that env instead, or stop and tell the user.

## Workflow

1. Load system config → identify repositories, `component`, `<env>`, and Metabase database.
2. Read relevant code in the listed repositories (primary first).
3. Query Loki with stream selector + filters from the system config.
4. Cross-check Metabase on the matching database for row state.
5. Summarize using the **output template** below.
6. For broader method (hypothesis, minimal repro): see **systematic-debugging** if present in the repo.

## Output template

Use this structure for every investigation report. Omit sections that do not apply; do not skip **Summary**, **Scope**, or **Evidence**.

```markdown
## Summary
<What failed, likely root cause, and confidence level in one short paragraph.>

## Scope
- **System:** <e.g. octopus, schema-system>
- **Environment:** <e.g. prd, prod, uat>
- **Component:** <Loki component>
- **Identifiers:** <order ref, line item id, job id, etc.>

## Evidence

### Logs
- **LogQL:** (one fenced block per query in the actual report)
- **Relevant lines:** <excerpts or key fields>

### Database
- **Connection:** <Metabase database name>
- **Queries run:** <SELECT statements used>
- **Row state:** <table, key columns, values that explain the failure>

### Code
- **Location:** <repo path and file/function>
- **Relevant behaviour:** <what the code did vs what was expected>

## Root cause
<Clear explanation tying evidence together.>

## Immediate remediation
<Operational actions to resolve **this specific case** without a code deploy — or "None identified".>

When recommending actions:
- Prefer existing runbook steps from `letterfestcode/docs` when they apply.
- For SQL: provide a **verification SELECT** first, then the **mutation** (UPDATE/INSERT/DELETE) with exact `WHERE` clauses and identifier values from evidence.
- State **who runs it** (Metabase SQL editor, psql, admin UI, Postman, etc.) and **which environment**.
- Call out **risks** (data loss, duplicate processing, reprints) and **how to verify** the issue is resolved.

Example (adapt to the case — do not copy blindly):

    -- Verify (run first)
    SELECT id, status, source_id FROM print_queue WHERE source_id = 'LF12345-67890';

    -- Remediation (run only if the row matches expectations above)
    UPDATE print_queue SET status = 'idle', attempts = 0 WHERE id = 12345 AND source_id = 'LF12345-67890';

## Code fix
<Include when asked to propose or implement a fix; otherwise omit.>

- **Change:** <what to change in the codebase>
- **Why:** <prevents recurrence>
- **Risk / rollout:** <flags, backfill, monitoring>
```

### Remediation vs code fix

| | Immediate remediation | Code fix |
| --- | --- | --- |
| **Purpose** | Unblock this order/job/incident now | Prevent recurrence |
| **Examples** | Reset queue status, requeue job, run admin regen endpoint, move Order Desk folder | Bug fix, validation, retry logic |
| **Delivered as** | SQL, CLI, UI steps, runbook links | PR / patch |
| **Executed by** | User or on-call (not via MCP) | Developer deploy |

## Adding a new system

Copy [systems/_template.md](systems/_template.md) to `systems/<system-name>.md` and fill in repositories, stream selectors, components, log fields, example LogQL, and Metabase database mapping.
