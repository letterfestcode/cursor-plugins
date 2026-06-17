# System config: octopus

## Scope

| Field | Value |
| --- | --- |
| System name | `octopus` |

## Repositories

| Repository | Role | Code paths | When to use |
| --- | --- | --- | --- |
| `letterfestcode/octopus` | Primary | `services/`, `apps/` | Backoffice API, fulfilment service, UIs |
| `letterfestcode/docs` | Related | `docs/docs/tech/on-call/runbooks/` | On-call runbooks |
| `letterfestcode/monitoring` | Related | `dashboards/octo/` | Loki labels, dashboards, alerts |

## Environments

| `<env>` | Meaning |
| --- | --- |
| `prd` | Production (default) |
| `uat` | UAT |

## Components

| Path | `component` | Notes |
| --- | --- | --- |
| `services/backoffice-api` | `backoffice-api` | |
| `services/fulfilment-service` | `fulfilment-service` | |
| `apps/backoffice-ui` | `backoffice-ui` | Loki: server-side logs only (SSR, API routes, Astro server handlers) |
| `apps/customer-portal` | `customer-portal` | Loki: server-side logs only |

## Stream selector

```logql
{product="octo", environment="<env>", component="<component>"}
```

| Placeholder | Value |
| --- | --- |
| `<env>` | `prd` (default) or `uat` |
| `<component>` | Leaf directory name under `services/` or `apps/` |

## Log format

Logs use **logfmt** (`key=value`). After the stream selector, pipe `| logfmt` and filter on fields.

### Common fields

| Field | Meaning |
| --- | --- |
| `lfInternalReference` | Order ref: `LF…`, `LFUS…`, `ETSY…`, `NOTHS…` |
| `lineItemId` | Marketplace line id — **not** internal `OrderLine.id` |
| `jobId` | Fulfilment job id |
| `level` | `error`, `warn`, `info`, … |
| `caller`, `class`, `function` | Log source (most lines) |

## Example LogQL queries

```logql
{product="octo", environment="prd", component="backoffice-api"} | logfmt | level="error"
{product="octo", environment="prd", component="fulfilment-service"} | logfmt | level="error"
{product="octo", environment="prd", component="backoffice-ui"} | logfmt | level="error"
{product="octo", environment="prd", component="backoffice-api"} | logfmt | lfInternalReference="LF12345"
{product="octo", environment="prd", component="backoffice-api"} | logfmt | lineItemId="1234567890"
{product="octo", environment="prd", component="fulfilment-service"} | logfmt | jobId="abc-uuid-here"
```

## Metabase databases

| Database | `<env>` | Use for |
| --- | --- | --- |
| `PRD \| Backoffice` | `prd` | `services/backoffice-api` |
| `UAT \| Backoffice` | `uat` | `services/backoffice-api` |
| `PRD \| Fulfilment` | `prd` | `services/fulfilment-service` |
| `UAT \| Fulfilment` | `uat` | `services/fulfilment-service` |

Query **`public`** schema tables only (e.g. `FROM "WorkflowLog"`). **`search` may return the same table name on multiple connections** — use the row for the env above, not a different env (see **Banned databases** in the main skill).

UIs under `apps/` do not have a separate Metabase database; order/line data is in **`<env> | Backoffice`** for that investigation.
