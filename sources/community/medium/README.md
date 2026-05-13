# Medium Community Source

This community source exposes a small, read-only slice of Medium's API through
Coral SQL: authenticated user profile info plus publication metadata.

Medium has archived its API documentation and states the API is no longer
supported. This source is best-effort and may break if Medium changes or
disables these endpoints.

## Setup

1. In your Medium account settings, create an Integration Token (Medium calls
   these "integration tokens").
2. Export it as `MEDIUM_TOKEN` or provide it via `--interactive` when adding the
   source.

```bash
MEDIUM_TOKEN='...' coral source add --file ./sources/community/medium/manifest.yaml
coral source test medium
```

The token should include the `basicProfile` scope. To query publications, the
token must include the `listPublications` scope.

## Example Queries

```sql
SELECT id, username, name, url
FROM medium.me;
```

```sql
SELECT id, name, url
FROM medium.publications
WHERE user_id = (SELECT id FROM medium.me LIMIT 1)
LIMIT 50;
```

```sql
SELECT user_id, role
FROM medium.publication_contributors
WHERE publication_id = '<publication-id>'
LIMIT 50;
```
