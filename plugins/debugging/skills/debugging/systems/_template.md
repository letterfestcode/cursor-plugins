# System config: `<system-name>`

Replace placeholders below. Copy this file to `<system-name>.md` when onboarding a new system.

## Scope

| Field | Value |
| --- | --- |
| System name | `<system-name>` — config key and how the agent recognises this product |

## Repositories

List every repo the agent may need for code context. At least one **primary** repo is required.

| Repository | Role | Code paths | When to use |
| --- | --- | --- | --- |
| `letterfestcode/<repo>` | Primary | e.g. `services/`, `apps/` | Application code for this system |
| `letterfestcode/docs` | Related | `docs/docs/tech/on-call/runbooks/` | On-call runbooks |
| `letterfestcode/monitoring` | Related | `dashboards/`, `collections/` | Loki labels, dashboards, alerts |

If the workspace is not the primary repo, search sibling clones under the team monorepo path or ask the user to open the correct repository.

## Environments

| `<env>` | Meaning |
| --- | --- |
| `prd` | Production (default) |
| `uat` | UAT / staging |

## Components

Map code paths (in the primary repo) to Loki `component` label values.

| Path | `component` | Notes |
| --- | --- | --- |
| `services/example-api` | `example-api` | |
| `apps/example-ui` | `example-ui` | Loki: server-side logs only (SSR, API routes) |

## Stream selector

```logql
{product="<product>", environment="<env>", component="<component>"}
```

| Placeholder | Value |
| --- | --- |
| `<product>` | Loki `product` label for this system |
| `<env>` | From environments table above (default: production row) |
| `<component>` | From components table above |

## Log format

Logs use **logfmt** (`key=value`). After the stream selector, pipe `| logfmt` and filter on fields.

### Common fields

| Field | Meaning |
| --- | --- |
| `level` | `error`, `warn`, `info`, … |
| `caller`, `class`, `function` | Log source |
| | _Add system-specific identifier fields here_ |

## Example LogQL queries

```logql
{product="<product>", environment="prd", component="<component>"} | logfmt | level="error"
```

## Metabase databases

Match database to **both** service and Loki `<env>`. Default `<env>` is production.

| Database | `<env>` | Use for |
| --- | --- | --- |
| `PRD \| Example` | `prd` | `services/example-api` |
| `UAT \| Example` | `uat` | `services/example-api` |

- Query **`public`** schema tables only (e.g. `FROM "SomeTable"`).
- Components without a dedicated Metabase DB: use the backing service database (e.g. Backoffice for UI investigations).
