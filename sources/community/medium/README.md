# Medium Community Source

Query your Medium profile, publications, and publication contributors using
authenticated access to Medium's API.

**Status:** Medium has archived its official API documentation. This source is
best-effort and may break if Medium changes or disables these endpoints.

## Setup

1. In your Medium account settings, create an Integration Token.
2. Export it as `MEDIUM_TOKEN` or provide it via `--interactive` when adding the
   source.

```bash
MEDIUM_TOKEN='...' coral source add --file ./sources/community/medium/manifest.yaml
coral source test medium
```

**Token Scopes:** The token must include the `basicProfile` scope to query your
profile. Add the `listPublications` scope to also query publications and
contributors.

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
