# Render source

Query core Render deployment data through Coral's HTTP source model.

## What this source covers

This source exposes five read-only tables:

- `render.services`: service inventory
- `render.deploys`: deploy history for one service
- `render.custom_domains`: custom domains for one service
- `render.projects`: project inventory
- `render.environments`: environments for one project

This first version is intentionally narrow. It does not include logs, metrics,
workflows, one-off jobs, or write operations.

## Authentication

The source requires a Render API key.

Inputs:

- `RENDER_API_BASE`
  - default: `https://api.render.com/v1`
- `RENDER_API_KEY`
  - required secret

Add the source:

```bash
cargo run --locked -p coral-cli -- source add render
```

To rotate or update the API key, run the same command again.

## Important pagination note

Render's list endpoints return arrays of wrapper objects like
`serviceWithCursor`, `projectWithCursor`, and `deployWithCursor`. Each row
carries its own `cursor` value.

Coral's current manifest model can expose those rows cleanly, but it cannot
automatically continue from the last row cursor without extra runtime support.

Because of that, this source supports **manual paging** through optional
`cursor` and `limit` filters on list-style tables. Each table also exposes a
`page_cursor` column so you can continue from the last row of a previous query.

## Table behavior

### `render.services`

Use this as the entry table for the source.

Optional filters:

- `cursor`
- `limit`

### `render.deploys`

Deploy history for one service.

Required filter:

- `service_id`

Optional filters:

- `cursor`
- `limit`

### `render.custom_domains`

Custom domains for one service.

Required filter:

- `service_id`

Optional filters:

- `cursor`
- `limit`

### `render.projects`

Project inventory.

Optional filters:

- `cursor`
- `limit`

### `render.environments`

Environment inventory for one project.

Required filter:

- `project_id`

Optional filters:

- `cursor`
- `limit`

## Validate the source

From the repo root:

```bash
cargo run --locked -p coral-cli -- source lint ./sources/render/manifest.yaml
make lint-sources
make docs-check
```

Then install and run the source tests:

```bash
cargo run --locked -p coral-cli -- source add render
cargo run --locked -p coral-cli -- source test render
```

## Inspect the installed shape

List the tables:

```bash
cargo run --locked -p coral-cli -- sql \
  "SELECT table_name FROM coral.tables WHERE schema_name = 'render' ORDER BY table_name"
```

Inspect columns:

```bash
cargo run --locked -p coral-cli -- sql \
  "SELECT table_name, column_name, type_name, is_required_filter \
   FROM coral.columns \
   WHERE schema_name = 'render' \
   ORDER BY table_name, ordinal_position"
```

Inspect inputs:

```bash
cargo run --locked -p coral-cli -- sql \
  "SELECT schema_name, key, kind, required, is_set, default_value \
   FROM coral.inputs \
   WHERE schema_name = 'render' \
   ORDER BY key"
```

## Example queries

List services:

```sql
SELECT id, name, type, slug, dashboard_url
FROM render.services
LIMIT 20;
```

Recent deploys for one service:

```sql
SELECT id, status, trigger, created_at, finished_at, commit__id, commit__message
FROM render.deploys
WHERE service_id = 'srv-1234567890'
LIMIT 20;
```

Custom domains for one service:

```sql
SELECT name, domain_type, verification_status, created_at
FROM render.custom_domains
WHERE service_id = 'srv-1234567890'
LIMIT 20;
```

Projects:

```sql
SELECT id, name, created_at, updated_at
FROM render.projects
LIMIT 20;
```

Environments for one project:

```sql
SELECT id, name, protected_status, network_isolation_enabled
FROM render.environments
WHERE project_id = 'prj-1234567890'
LIMIT 20;
```

Manual paging example:

```sql
SELECT page_cursor, id, name
FROM render.services
LIMIT 20;
```

Take the last non-null `cursor` from that result, then continue:

```sql
SELECT page_cursor, id, name
FROM render.services
WHERE cursor = 'cur-previous-page-end'
LIMIT 20;
```
