---
name: coral
description: "Query APIs, files, and live sources using Coral SQL. Use when the user asks about data from GitHub, Slack, Linear, Datadog, Sentry, or other connected sources."
---

# Coral CLI Skill

Use this skill to answer data questions by querying connected sources with `coral sql`.

## Discovery-First Workflow

Before writing any query, follow these steps:

1. **List visible tables, descriptions, guides, and required filters:**

```sql
SELECT schema_name, table_name, description, required_filters, guide
FROM coral.tables
ORDER BY schema_name, table_name;
```

2. **Read the `guide` column** — it contains per-table query patterns and gotchas. Use it.

3. **Find tables by substring within a known schema when needed:**

```sql
SELECT schema_name, table_name, description, required_filters
FROM coral.tables
WHERE schema_name = '<source>' AND table_name LIKE '%<term>%'
ORDER BY schema_name, table_name;
```

4. **Use `DESCRIBE <source>.<table>` or `SHOW COLUMNS FROM <source>.<table>` only as quick shape shortcuts.**

5. **Inspect source inputs when config affects the query or answer.** This matters when you need source-specific values such as Datadog's `DD_SITE` or Sentry's `SENTRY_ORG` to build absolute links, explain account scope, or debug missing configuration.

```sql
SELECT key, kind, value, default_value, hint, required, is_set
FROM coral.inputs
WHERE schema_name = '<source>'
ORDER BY key;
```

6. **Use `coral.columns` as canonical column metadata** for column types, descriptions, virtual columns, and required-filter markers. Queries against tables with required filters will fail without the corresponding WHERE clauses.

```sql
SELECT column_name, data_type, is_required_filter, is_virtual, description
FROM coral.columns
WHERE schema_name = '<source>' AND table_name = '<table>'
ORDER BY ordinal_position;
```

7. **Then query:**

```bash
coral sql "SELECT <columns> FROM <source>.<table> WHERE <required_filters> LIMIT 10"
```

## Query Guidance

- **Virtual columns:** Filter-only columns accepted in WHERE clauses but returning NULL in results. Check `is_virtual` in `coral.columns`.
- **`coral.inputs`:** Use it to inspect per-source variables and secrets before making assumptions about URLs, org names, regions, or other source config. Secret rows always return `value = NULL`; use `is_set` to confirm whether a secret is configured.
- **Cross-source joins:** Standard SQL JOINs work across source schemas. Cross-source joins execute in memory after source scans complete.
