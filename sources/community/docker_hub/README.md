# Docker Hub Source

This source exposes public Docker Hub repository and image tag metadata through
Coral SQL.

## Scope

This initial source is read-only and uses Docker Hub's public API endpoints. It
is intended for public repository discovery, repository metadata checks, and tag
inspection.

Private repositories and authenticated account inventory are intentionally out
of scope for this manifest-only version. Docker Hub personal access tokens are
exchanged for bearer tokens through an auth flow; supporting that cleanly would
need a custom authenticator instead of plain source-spec header templating.

## Authentication

No credentials are required. The source only queries public Docker Hub
repository and tag metadata.

Some repository detail fields, such as `permissions`, `affiliation`, and
`has_starred`, are request-context fields from Docker Hub's response. Because
this source does not send credentials, those fields reflect Docker Hub's
unauthenticated response context.

## Rate Limits

Docker Hub rate limits API requests. Responses include rate-limit headers such
as `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset`.

The manifest uses conservative page sizes for list endpoints. Add required
filters such as `namespace`, `repository`, and `query` to keep requests focused.

## Tables

| Table | Description |
| --- | --- |
| `search_repositories` | Search public Docker Hub repositories by keyword. |
| `repositories` | List public repositories in a namespace, such as `library`. |
| `repository_details` | Fetch detailed metadata for one public repository. |
| `tags` | List image tags for one public repository. |
| `tag_details` | Fetch metadata for one repository tag. |

## Example Queries

```sql
SELECT repo_name, star_count, pull_count
FROM docker_hub.search_repositories
WHERE query = 'alpine'
LIMIT 10;
```

```sql
SELECT name, pull_count, last_updated
FROM docker_hub.repositories
WHERE namespace = 'library'
ORDER BY pull_count DESC
LIMIT 10;
```

```sql
SELECT name, digest, tag_last_pushed, full_size
FROM docker_hub.tags
WHERE namespace = 'library' AND repository = 'alpine'
LIMIT 20;
```

```sql
SELECT digest, media_type, images
FROM docker_hub.tag_details
WHERE namespace = 'library' AND repository = 'alpine' AND tag = 'latest';
```
