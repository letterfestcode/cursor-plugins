# System config: schema-system

## Scope

| Field | Value |
| --- | --- |
| System name | `schema-system` |

## Repositories

| Repository | Role | Code paths | When to use |
| --- | --- | --- | --- |
| `letterfestcode/schema-system` | Primary | `src/` | Application code for both components |

## Environments

| `<env>` | Meaning |
| --- | --- |
| `prod` | Production (default) — note: `prod`, not `prd` |

There is no separate UAT Loki stream for legacy products in monitoring config.

## Components

Map Loki `component` label values to code in `src/`. Both components share general libraries under `src/lib/`.

| Path | `component` | Notes |
| --- | --- | --- |
| `src/` | `artwork` | artwork.letterfest.com — order import, print queue, Dropbox generation |
| `src/bookcheckers/` | `bookcheckers` | bookcheckers.letterfest.com — also uses shared `src/lib/` |

## Stream selector

```logql
{product="legacy", environment="<env>", component="<component>"}
```

| Placeholder | Value |
| --- | --- |
| `<env>` | `prod` (default) |
| `<component>` | `artwork` or `bookcheckers` |

PHP logs may also filter `| logfmt | program="php-fpm"` or nginx-parsed PHP messages.

## Log format

Logs use **logfmt** (`key=value`) where the file has been migrated to the new `Logger` standard. Many files still use legacy logging — **do not rely on consistent structured fields**.

When logfmt fields are present, you may see `level`, `caller`, `class`, `function`, `msg`, `error`, `cause`, and occasionally `lfInternalReference`, `sourceId`, or `jobId` — but availability varies by file.

**Default approach:** narrow by `component` and `level="error"`, then use `|~` on order references, class names, or exception text.

## Example LogQL queries

```logql
{product="legacy", environment="prod", component="artwork"} | logfmt | level="error"
{product="legacy", environment="prod", component="bookcheckers"} | logfmt | level="error"
{product="legacy", environment="prod", component="artwork"} | logfmt | program="php-fpm" | level="error"
{product="legacy", environment="prod", component="artwork"} |~ "LF12345"
{product="legacy", environment="prod", component="bookcheckers"} |~ "LF12345"
```

## Metabase databases

Both components use the same connection. Default `<env>` is `prod`.

| Database | `<env>` | Use for |
| --- | --- | --- |
| `PRD \| Schema System` | `prod` | `artwork`, `bookcheckers` |

Key tables:

| Table | Use for |
| --- | --- |
| `print_queue` | Print job status (`failed`, retries); search by `source_id`, `order_ref`, or `line_item_id` |
| `orders` | Etsy Order Desk ID ↔ Etsy order ref / line item id |
| `misc_jobs_queue` | Bookchecker async job queue |
| `octopus_message_queue` | Postbacks to Octopus |
| `illustration_artwork_jobs` | Illustration CSV / template generation queue |

Query only tables on `PRD | Schema System`. If `search` returns the same table on multiple connections, use this connection for `prod` (see **Banned databases** in the main skill).
