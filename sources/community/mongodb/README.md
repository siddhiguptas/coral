# MongoDB Atlas

Query MongoDB Atlas infrastructure and observability metadata from the Atlas Administration API.

This source treats Atlas as a cloud database infrastructure platform, not as a generic MongoDB document query engine.

## Setup

### Create an Atlas Service Account Token

Create a MongoDB Atlas service account with the least-privilege roles needed for the tables you want to query. For most inventory and observability tables, use Organization Member plus Project Read Only on the relevant projects.

Add your current IP address or CIDR range to the service account API access list. Atlas rejects Admin API requests from IPs that are not on this list.

Generate an OAuth access token from the service account client ID and secret:

```bash
ACCESS_TOKEN=$(curl -fsS --request POST https://cloud.mongodb.com/api/oauth/token \
  --header "Authorization: Basic $(printf '%s' "${MONGODB_ATLAS_CLIENT_ID}:${MONGODB_ATLAS_CLIENT_SECRET}" | base64 | tr -d '\n')" \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --header "Accept: application/json" \
  --data "grant_type=client_credentials" | jq -r '.access_token')
```

Atlas access tokens are valid for 1 hour.

### Add the Source

```bash
MONGODB_ATLAS_ACCESS_TOKEN="$ACCESS_TOKEN" coral source add mongodb
```

When prompted interactively, provide the token as `MONGODB_ATLAS_ACCESS_TOKEN`.

## Tables

### `organizations`

Atlas organizations visible to the service account.

**Useful for:**
- Validating credentials
- Discovering `org_id` values
- Understanding which organizations the service account can inspect

### `projects`

Atlas projects visible to the service account. Atlas API paths also call projects "groups".

**Useful for:**
- Project inventory
- Finding `project_id` values for project-scoped tables
- Mapping projects to organizations

### `clusters`

Cluster inventory for a single Atlas project.

**Requires:** `project_id`

**Useful for:**
- Database infrastructure inventory
- Region and cloud provider review
- MongoDB version and cluster state inspection
- Detecting stale, paused, or unexpectedly large clusters

This table does not connect to MongoDB clusters or query collection documents.

### `database_users`

Database user metadata for a single Atlas project.

**Requires:** `project_id`

**Security note:** This table intentionally does not expose passwords, generated credentials, connection strings, tokens, or secrets.

**Useful for:**
- Database user inventory
- Role and scope review
- Authentication mode inspection

### `alerts`

Atlas alerts for a single project.

**Requires:** `project_id`

Optional filters:
- `status`
- `event_type`

**Useful for:**
- Incident summaries
- Noisy cluster detection
- Operational analysis
- Outage and alert timeline review

### `project_events`

Atlas project activity events.

**Requires:** `project_id`

Optional filters:
- `event_type`
- `min_date`
- `max_date`

**Useful for:**
- Infrastructure change history
- Security and governance timelines
- Correlating alerts with project activity

This table exposes event metadata. It does not fetch raw audit log files.

### `audit_config`

Database auditing configuration for a single Atlas project.

**Requires:** `project_id`

**Permission note:** Atlas requires Project Owner for this endpoint. The endpoint is not available for M0, M2, M5, or serverless clusters.

**Useful for:**
- Checking whether auditing is enabled
- Reviewing audit filter configuration
- Governance posture summaries

This table does not fetch raw audit log streams.

### `processes`

Atlas process inventory for a single project.

**Requires:** `project_id`

**Useful for:**
- Monitoring host inventory
- Replica set and shard topology review
- Finding process IDs for future metrics queries

## Authentication

This source uses Atlas Administration API OAuth bearer authentication:

```text
Authorization: Bearer <MONGODB_ATLAS_ACCESS_TOKEN>
Accept: application/vnd.atlas.2025-03-12+json
```

MongoDB recommends service accounts for Atlas Administration API authentication. Legacy public/private API keys use HTTP Digest authentication; Coral's built-in source spec auth does not currently implement Digest auth or OAuth token refresh, so v1 expects a pre-minted bearer token.

Do not put the service account client secret in the manifest or README. Use it only to mint a short-lived access token, then pass that token to Coral as `MONGODB_ATLAS_ACCESS_TOKEN`.

## Limits

- This source exposes read-only Atlas Administration API endpoints only.
- Atlas OAuth access tokens expire after 1 hour and must be refreshed outside Coral for now.
- Project-scoped tables require `project_id` to avoid broad, expensive scans.
- Raw MongoDB document querying is out of scope.
- Collection scans are out of scope.
- Cluster mutations, provisioning, pause/resume, and configuration updates are out of scope.
- Raw logs and compressed audit log streams are out of scope.
- Backups, restores, private networking, access lists, and network configuration are out of scope for v1.
- App Services, Data API, stream processing, and Atlas Search index management are separate product surfaces and are out of scope for v1.

## Example Queries

### List clusters by cloud provider and region

```sql
SELECT provider_name, region_name, COUNT(*) AS cluster_count
FROM mongodb.clusters
WHERE project_id = 'your-project-id'
GROUP BY provider_name, region_name
ORDER BY cluster_count DESC
```

### Find paused or unhealthy clusters

```sql
SELECT cluster_name, state_name, paused, mongo_version, disk_size_gb
FROM mongodb.clusters
WHERE project_id = 'your-project-id'
  AND (paused = true OR state_name <> 'IDLE')
ORDER BY cluster_name
```

### Review active alerts

```sql
SELECT alert_id, event_type, status, cluster_name, created_at, updated_at
FROM mongodb.alerts
WHERE project_id = 'your-project-id'
  AND status = 'OPEN'
ORDER BY created_at DESC
LIMIT 50
```

### Summarize recent project activity

```sql
SELECT event_type, username, target_type, target_name, created_at
FROM mongodb.project_events
WHERE project_id = 'your-project-id'
ORDER BY created_at DESC
LIMIT 100
```

### Check database users and roles

```sql
SELECT username, auth_database, roles, scopes
FROM mongodb.database_users
WHERE project_id = 'your-project-id'
ORDER BY username
```

### Inspect audit posture

```sql
SELECT auditing_enabled, configuration_type, audit_authorization_success, audit_filter
FROM mongodb.audit_config
WHERE project_id = 'your-project-id'
```

## Notes

- Use `mongodb.projects` first to find project IDs.
- Use `mongodb.organizations` first to confirm the service account can access the expected organization.
- JSON columns preserve nested Atlas metadata without flattening every provider-specific field.
- Timestamps are stored as proper Timestamp columns derived from Atlas ISO 8601 strings.
