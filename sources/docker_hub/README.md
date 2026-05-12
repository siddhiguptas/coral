# Docker Hub Source

This source exposes public Docker Hub repository and image tag metadata through
Coral SQL.

## Scope

The v1 source is read-only and uses Docker Hub's public API endpoints. It does
not require credentials and is intended for public repository discovery,
repository metadata checks, and tag inspection.

Private repositories and authenticated account inventory are intentionally out
of scope for this manifest-only version. Docker Hub personal access tokens are
exchanged for bearer tokens through an auth flow; supporting that cleanly would
need a custom authenticator instead of plain source-spec header templating.

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
