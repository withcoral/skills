# HTTP Source Checklist

Use this checklist when authoring an HTTP-backed Coral source.

## Start Small

- Begin with one collection endpoint.
- Add only a few columns first.
- Import and query before expanding coverage.

## Source Header

Include:

- `name`
- `version`
- `dsl_version`
- `backend: http`
- `base_url`
- `auth`

Use:

- variables for non-secret configuration such as API base URLs
- secrets for API keys, tokens, and client secrets

## Table Design

- Prefer one table per collection endpoint.
- Add detail routes only when item fetches are actually needed.
- Keep table names stable and SQL-friendly.
- Preserve provider semantics when filter behavior matters.

## Response Extraction

- Set `rows_path` to the array Coral should read as rows.
- Use the default direct row strategy unless the payload shape requires something else.
- Keep the first pass simple; add special handling only after validating the payload shape.

## Filters

- Mark filters as required only when the upstream API requires them.
- Use seed queries to discover real IDs for child tables.
- If a table keeps failing with a missing required filter, inspect `coral.columns` and match the exact filter name.

## Pagination

Prefer explicit pagination when the provider pattern is known.

- `limit` + `offset` APIs:
  - use offset pagination
- numbered-page APIs:
  - use page pagination
- cursor/token APIs:
  - use cursor pagination

Do not treat `COUNT(*)` as sufficient pagination proof. Fetch actual rows and confirm that results extend beyond one page.

## Validation Loop

Use this loop while iterating:

```sh
coral source add --file ./my-source.yaml
coral source test my_source
coral sql "SELECT table_name FROM coral.tables WHERE schema_name = 'my_source'"
coral sql "SELECT table_name, column_name, is_required_filter FROM coral.columns WHERE schema_name = 'my_source' ORDER BY table_name, column_name"
```

Then run targeted table queries with real filters.
