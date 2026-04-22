---
name: coral-create-source-spec
description: Create or update a Coral source spec YAML for a custom HTTP API or local dataset. Use when authoring a standalone source for `coral source add --file`, or when adapting that spec into a bundled source in the Coral repo.
---

# Create Source Spec

Use this skill when the task is to author or repair a Coral source spec.

## Goal

Produce a valid, queryable Coral source spec that works with:

- `coral source add --file <path>`
- `coral source test <name>`
- `coral sql`
- `coral.tables` and `coral.columns`
- `coral.inputs` for source variables and secrets

## Default Mode

Default to standalone source authoring for external developers.

That means:

- create a YAML source spec file
- add it with `coral source add --file`
- validate by querying it
- iterate until the shape is correct

Only switch to repo-bundled layout when the user is explicitly editing the Coral repo.

## Output Modes

- External authoring:
  - create a standalone source spec such as `./my-source.yaml`
  - validate with `coral source add --file ./my-source.yaml`
- Coral repo contribution:
  - write the source spec to `sources/<name>/manifest.yaml`
  - validate with `coral source test <name>` and repo checks

## Workflow

1. Read the provider API docs or inspect the local dataset.
2. Start with one small table and a few columns.
3. Define:
   - source metadata
   - backend
   - base URL or file location
   - auth
   - variables and secrets
   - tables
   - filters
   - response extraction
   - pagination
   - typed columns
4. Import the source:
   - `coral source add --file <path>`
5. Validate the imported shape:
   - `coral source test <name>`
   - inspect `coral.tables`
   - inspect `coral.columns`
   - inspect `coral.inputs` to verify variables, secrets, defaults, hints, and required flags
6. Query representative tables with `coral sql`.
7. Refine the spec and repeat.

## Authoring Rules

- Start small and expand table coverage incrementally.
- Use the source manifest schema as both inspiration for authoring and validation of structure: https://github.com/withcoral/coral/blob/main/crates/coral-spec/src/schema/source_manifest.schema.json
- Use source variables for non-secret configuration.
- Use source secrets for credentials.
- Keep table names stable and SQL-friendly.
- Mark filters as required only when the API truly requires them.
- Prefer explicit pagination when the API shape is known.
- Verify pagination with actual row fetches, not only `COUNT(*)`.

## Metadata UX Rules

Use these rules for top-level source metadata so source discovery and setup are consistent.

### `description`

- Start with `Query ...`.
- Make the first sentence capability-first: list the key entities users can query.
- Preferred template:
  - `Query <entities> from <Provider> (<Cloud or self-hosted when relevant>).`
- Keep `description` focused on data coverage, not setup steps.
- Do not use vague phrasing such as:
  - `REST API v3`
  - `OpenAPI provider`
  - `... and more`
- Move auth/setup/permission details to input hints, not description text.

### Input hints (`inputs.<KEY>.hint`)

Each hint should tell the user:

- what value is expected
- how to obtain it
- minimum scope/permission guidance
- one concrete format/example when useful

Specific guidance:

- For URL/base inputs:
  - say what the default means
  - include at least one concrete example
  - include self-hosted guidance when supported
- For secrets:
  - name the exact credential type (API key, PAT, application key, etc.)
  - include format constraints when relevant (for example, token prefixes)
  - include least-privilege scope guidance
- For derived secrets (for example Basic auth blobs):
  - include a short shell example (for example a Base64 command)
- Prefer stable documentation links.
  - Use official docs links and stable settings pages.
  - Avoid brittle click-path instructions as the primary guidance.

Keep hints concise and directly actionable.

## Validation Loop

Use this loop during authoring:

```sh
coral source add --file ./my-source.yaml
coral source test my_source
coral sql "SELECT * FROM coral.tables WHERE schema_name = 'my_source'"
coral sql "SELECT * FROM coral.columns WHERE schema_name = 'my_source'"
coral sql "SELECT key, kind, value, default_value, hint, required, is_set FROM coral.inputs WHERE schema_name = 'my_source' ORDER BY key"
```

Then run targeted table queries until the source behaves correctly.

## HTTP Sources

For HTTP-backed sources:

- define `backend: http`
- define `base_url`
- define auth headers
- define request path, query, and body only where needed
- define response `rows_path`
- define pagination explicitly when the provider pattern is known
- define typed columns

Read `references/http-source-checklist.md` when you need table-shape and pagination guidance.

## Local Data Sources

For local file-backed sources:

- define the file backend
- define the source location
- define file selection patterns if applicable
- define typed columns

## Deliverable

Report:

- source spec path
- import command used
- validation commands run
- assumptions made
- blocked or unverified endpoints
