# Firebase

Query Firebase Management API data from Coral.

This community source currently covers Firebase project and app inventory,
plus project-to-Google Analytics linkage details:

- `firebase.projects`
- `firebase.project`
- `firebase.android_apps`
- `firebase.ios_apps`
- `firebase.web_apps`
- `firebase.project_apps`
- `firebase.analytics_details`

It intentionally starts with the Firebase Management REST API. Firestore and
Realtime Database expose user-defined data shapes, so they should be designed as
a follow-up source or as separate tables once the desired query shape is clear.

## Authentication

Set `FIREBASE_ACCESS_TOKEN` to a Google OAuth 2.0 access token that can call the
Firebase Management API.

For local testing with Google Cloud CLI:

```powershell
$env:FIREBASE_ACCESS_TOKEN = gcloud auth print-access-token
```

Use a token with `https://www.googleapis.com/auth/firebase.readonly` for
read-only access. `https://www.googleapis.com/auth/cloud-platform.read-only`
can also work when your Google Cloud permissions are managed more broadly.

## Example Queries

```sql
SELECT * FROM firebase.projects LIMIT 10;

SELECT *
FROM firebase.android_apps
WHERE project_id = 'my-firebase-project'
LIMIT 10;

SELECT *
FROM firebase.project_apps
WHERE project_id = 'my-firebase-project'
LIMIT 10;

SELECT *
FROM firebase.project_apps
WHERE project_id = 'my-firebase-project'
  AND filter = 'platform=ANDROID'
LIMIT 10;

SELECT *
FROM firebase.analytics_details
WHERE project_id = 'my-firebase-project'
LIMIT 1;
```

## Validation

```powershell
coral source lint sources/community/firebase/manifest.yaml
$env:FIREBASE_ACCESS_TOKEN = gcloud auth print-access-token
coral source add --file sources/community/firebase/manifest.yaml
coral source test firebase
coral sql "SELECT * FROM coral.tables WHERE schema_name = 'firebase'"
coral sql "SELECT * FROM coral.columns WHERE schema_name = 'firebase'"
coral sql "SELECT key, kind, required, is_set FROM coral.inputs WHERE schema_name = 'firebase'"
```
