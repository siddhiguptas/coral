# Postman

Query workspaces, collections, requests, environments, and monitors from Postman.

## Setup

### Get Your API Key

1. Visit [Postman API Keys](https://web.postman.co/settings/me/api-keys)
2. Generate a new API key
3. Copy the key

### Add the Source

```bash
coral source add postman
```

When prompted, provide your Postman API key.

## Tables

### `me`
Returns the current authenticated user information. Use this to validate your API key configuration.

**Useful for:**
- Verifying authentication
- Getting user metadata
- Account identification

### `workspaces`
Top-level organizational entities in Postman. All collections, environments, and monitors are grouped within workspaces.

**Useful for:**
- Workspace inventory
- Organization structure
- Workspace metadata queries

### `collections`
Collections are grouped API workflows, requests, tests, and documentation. This is one of the most important Postman entities.

**Useful for:**
- API inventory analysis
- Collection browsing
- Request organization
- Finding collections by owner or workspace

**Example:**
```sql
SELECT name, owner_id, workspace_id 
FROM postman.collections 
WHERE workspace_id = 'your-workspace-id'
```

### `collection_requests`
Request-level metadata extracted from collections. This table exposes endpoints, HTTP methods, authentication configs, and test scripts.

**Requires:** `collection_uid` filter

**Useful for:**
- API endpoint discovery
- HTTP method analytics
- Understanding request structure
- Finding endpoints by method

**Example:**
```sql
SELECT method, COUNT(*) as request_count
FROM postman.collection_requests
WHERE collection_uid = 'your-collection-uid'
GROUP BY method
```

### `environments`
Variable groups used across testing and deployment workflows. Environments allow parameterization of requests.

**Useful for:**
- Environment inventory
- Variable structure
- Testing configuration
- Deployment setup tracking

### `monitors`
Continuous API monitors that test health and reliability. Monitors run on schedules and report failures.

**Useful for:**
- Monitor health dashboards
- Failure tracking
- Reliability analytics
- API governance

**Example:**
```sql
SELECT name, status, failure_count
FROM postman.monitors
WHERE status = 'running'
ORDER BY failure_count DESC
```

## Authentication

The source uses Postman API Key authentication. Your API key is sent via the `X-API-Key` header.

**Minimum scopes required:**
- workspaces.read
- collections.read
- environments.read
- monitors.read

## Limits

- Postman API is rate-limited
- Results are paginated (default 25 items per page)
- Use `LIMIT` and `OFFSET` to control pagination in queries

## Example Queries

### Get method distribution across all requests in a collection

```sql
SELECT method, COUNT(*) as request_count
FROM postman.collection_requests
WHERE collection_uid = 'your-collection-uid'
GROUP BY method
ORDER BY request_count DESC
```

### Find all failing monitors

```sql
SELECT name, failure_count, updated_at
FROM postman.monitors
WHERE status = 'running'
ORDER BY failure_count DESC
LIMIT 10
```

### Inventory collections by workspace

```sql
SELECT workspace_id, COUNT(*) as collection_count
FROM postman.collections
GROUP BY workspace_id
ORDER BY collection_count DESC
```

### Find collections with no owner specified

```sql
SELECT name, created_at
FROM postman.collections
WHERE owner_id IS NULL
```

## Notes

- The `collection_requests` table requires a `collection_uid` to avoid expensive full scans
- JSON columns (variables, auth, headers, scripts) contain nested structures
- Timestamps are in ISO 8601 format
- Secret variable values are not exposed for security reasons
