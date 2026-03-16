# Identity & Authentication

Marketo Engage uses OAuth 2.0 client credentials for API authentication. The Identity API issues access tokens; all other API calls require a valid token.

**Base URL:** `https://{munchkin-id}.mktorest.com`

---

## Setup

1. In Marketo Admin, go to **Users & Roles > Roles** and create or select a role with the needed API permissions.
2. Go to **Users & Roles > Users** and create an **API-Only** user assigned to that role.
3. Go to **Admin > Integration > LaunchPoint** and create a **New Service** (type: Custom). Select the API-only user.
4. Click **View Details** on the service to get your `client_id` and `client_secret`.
5. Go to **Admin > Integration > Web Services** to find your REST API endpoint URL and Identity URL.

The endpoint URL looks like: `https://123-ABC-456.mktorest.com/rest`
The identity URL looks like: `https://123-ABC-456.mktorest.com/identity`

## Get Access Token

```
GET /identity/oauth/token?grant_type=client_credentials&client_id={id}&client_secret={secret}
```

Also supports `POST` with the same query parameters.

| Param | Required | Description |
|-------|----------|-------------|
| grant_type | Yes | Must be `client_credentials` |
| client_id | Yes | Client ID from LaunchPoint service |
| client_secret | Yes | Client Secret from LaunchPoint service |

```bash
curl -s "https://123-ABC-456.mktorest.com/identity/oauth/token?\
grant_type=client_credentials&\
client_id=abc123-def456-ghi789&\
client_secret=XXXXXXXXXXXXXXXXXXXXXXXX"
```

**Response:**

```json
{
  "access_token": "cdf01657-110d-4155-99a7-f986b2ff13a0:int",
  "token_type": "bearer",
  "expires_in": 3599,
  "scope": "api-only-user@example.com"
}
```

| Field | Description |
|-------|-------------|
| access_token | Bearer token to pass in `Authorization` header for all subsequent calls |
| token_type | Always `bearer` |
| expires_in | Seconds until expiry (max 3600 = 1 hour) |
| scope | The API-only user email associated with the service |

## Using the Token

Include the token in all API requests:

```bash
curl -s -H "Authorization: Bearer cdf01657-110d-4155-99a7-f986b2ff13a0:int" \
  "https://123-ABC-456.mktorest.com/rest/v1/leads.json?filterType=email&filterValues=test@example.com"
```

## Token Lifecycle

- Tokens expire after **3,600 seconds** (1 hour).
- Identity calls are **not** counted against the daily API quota (50,000 calls/day).
- You can request a new token at any time; existing tokens remain valid until they expire.
- Cache the token and reuse it until you get a `601` or `602` error, then re-authenticate.

### Token Refresh Pattern

```bash
#!/bin/bash
MARKETO_HOST="https://123-ABC-456.mktorest.com"
CLIENT_ID="your-client-id"
CLIENT_SECRET="your-client-secret"

get_token() {
  curl -s "${MARKETO_HOST}/identity/oauth/token?\
grant_type=client_credentials&\
client_id=${CLIENT_ID}&\
client_secret=${CLIENT_SECRET}" | jq -r '.access_token'
}

TOKEN=$(get_token)

response=$(curl -s -H "Authorization: Bearer $TOKEN" \
  "${MARKETO_HOST}/rest/v1/leads.json?filterType=email&filterValues=test@example.com")

success=$(echo "$response" | jq -r '.success')
if [ "$success" = "false" ]; then
  error_code=$(echo "$response" | jq -r '.errors[0].code')
  if [ "$error_code" = "601" ] || [ "$error_code" = "602" ]; then
    TOKEN=$(get_token)
    response=$(curl -s -H "Authorization: Bearer $TOKEN" \
      "${MARKETO_HOST}/rest/v1/leads.json?filterType=email&filterValues=test@example.com")
  fi
fi

echo "$response"
```

## Error Responses

**Invalid credentials:**

```json
{
  "error": "invalid_client",
  "error_description": "Bad client credentials"
}
```

**Token errors on API calls:**

| Code | Message | Action |
|------|---------|--------|
| 601 | Access token invalid | Re-authenticate |
| 602 | Access token expired | Re-authenticate |

## Permissions Reference

Each API endpoint requires specific permissions on the API-only user's role. Common permission groups:

| Permission | Grants Access To |
|------------|-----------------|
| Read-Only Lead | Get leads, list membership, activities |
| Read-Write Lead | Create/update/delete leads, program members |
| Read-Only Assets | Get emails, forms, landing pages, programs, snippets, folders |
| Read-Write Assets | Create/update/delete assets |
| Approve Assets | Approve/unapprove drafts |
| Read-Only Activity | Get activities, activity types, paging tokens |
| Read-Write Activity | Add custom activities |
| Read-Write Activity Metadata | Create/update/delete custom activity types |
| Read-Only Campaign | Get campaigns |
| Read-Write Campaign | Trigger/schedule campaigns |
| Read-Only Custom Object | Get custom objects |
| Read-Write Custom Object | Sync/delete custom objects |
| Read-Write Custom Object Type | Create/update/delete custom object schemas |
| Read-Only Company | Get companies |
| Read-Write Company | Sync/delete companies |
| Read-Only Named Account | Get named accounts |
| Read-Write Named Account | Sync/delete named accounts |
| Access User Management Api | User CRUD, roles, workspaces |
| Read-Write Schema Custom Field | Manage lead/company/opportunity field schemas |

## Rate Limits & Quotas

| Limit | Value |
|-------|-------|
| Daily API calls | 50,000 (resets midnight Central Time) |
| Concurrent API calls | 100 |
| Bulk extract jobs (concurrent) | 10 |
| Bulk import jobs (concurrent) | 10 |
| Bulk extract file size | 500MB max |
| Bulk import file size | 10MB max |
| Max batch size per call | 300 records |
| Max lead list per sync call | 300 leads |

When rate limited, the API returns error code `606` (Rate limit exceeded) or `607` (Daily quota reached).
