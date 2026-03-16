---
name: api
description: "Adobe Marketo Engage REST API for marketing automation — identity/auth, leads, emails, forms, landing pages, programs, smart campaigns, custom objects, bulk import/export, and user management"
metadata:
  languages: "rest"
  versions: "1.0"
  revision: 1
  updated-on: "2026-03-15"
  source: community
  tags: "marketo,adobe,marketing-automation,rest,api,leads,emails,forms,campaigns"
---
# Marketo Engage REST API

Marketo Engage exposes a REST API over HTTPS using OAuth 2.0 client credentials. The base URL is unique to your Marketo subscription and follows the pattern `https://{munchkin-id}.mktorest.com`. Find your Munchkin ID and endpoint URL in **Admin > Web Services**.

All request and response bodies are JSON (`application/json`) unless noted. The API enforces a daily quota of 50,000 API calls and a rate limit of 100 concurrent calls.

## Authentication

Every API call (except the token request itself) must include an `Authorization: Bearer <access_token>` header. Tokens are obtained from the Identity endpoint and expire after 3,600 seconds (1 hour). Token requests are **not** counted against the daily quota.

```bash
# Obtain an access token
curl -s "https://{munchkin-id}.mktorest.com/identity/oauth/token?\
grant_type=client_credentials&\
client_id=YOUR_CLIENT_ID&\
client_secret=YOUR_CLIENT_SECRET"
```

Response:

```json
{
  "access_token": "cdf01657-110d-4155-99a7-f986b2ff13a0:int",
  "token_type": "bearer",
  "expires_in": 3599,
  "scope": "api-only-user@example.com"
}
```

**Setup:** Create a custom service in **Admin > LaunchPoint > New Service** (type "Custom"), then view details to get `client_id` and `client_secret`. Assign an API-only user with the appropriate permissions.

## Common Request Pattern

All REST endpoints return a standard envelope:

```json
{
  "requestId": "e42b#14272d07d78",
  "success": true,
  "result": [ ... ]
}
```

On error:

```json
{
  "requestId": "e42b#14272d07d78",
  "success": false,
  "errors": [
    { "code": "601", "message": "Access token invalid" }
  ]
}
```

### Common Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| 601 | Access token invalid | Re-authenticate |
| 602 | Access token expired | Re-authenticate |
| 606 | Rate limit exceeded | Back off and retry |
| 607 | Daily quota reached | Wait until quota resets (midnight CT) |
| 608 | API temporarily unavailable | Retry with backoff |
| 611 | System error | Retry with backoff |
| 703 | Feature not enabled | Contact Marketo admin |

### Paging

List endpoints use `nextPageToken` for cursor-based pagination. Pass the token from the previous response to get the next page. The default and max `batchSize` is 300.

```bash
# First page
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/v1/leads.json?filterType=email&filterValues=a@example.com,b@example.com&batchSize=2"

# Next page (use nextPageToken from response)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/v1/leads.json?filterType=email&filterValues=a@example.com,b@example.com&batchSize=2&nextPageToken=TOKENFROMRESPONSE"
```

## API Domains

The Marketo API is organized into five service areas:

| Domain | Base Path | Description |
|--------|-----------|-------------|
| **Identity API** | `/identity/` | OAuth 2.0 token endpoint, authentication |
| **Asset API** | `/rest/asset/v1/` | Emails, email templates, forms, landing pages, snippets, programs, smart campaigns, channels, folders, tokens, files |
| **Lead Database API** | `/rest/v1/` | Leads, companies, opportunities, activities, custom objects, custom activity types, named accounts, static lists, transactional email send |
| **Bulk API** | `/bulk/v1/` | High-volume import/export for leads, activities, custom objects, program members |
| **User Management** | `/userservice/management/v1/` | Users, roles, workspaces |

## Quick Examples

### Create/Update a Lead

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/v1/leads.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createOrUpdate",
    "lookupField": "email",
    "input": [
      {
        "email": "john@example.com",
        "firstName": "John",
        "lastName": "Doe",
        "company": "Acme Corp"
      }
    ]
  }'
```

### Get an Email Asset by ID

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/email/1234.json"
```

### Browse Forms

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/forms.json?maxReturn=20&offset=0"
```

### Trigger a Smart Campaign

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/v1/campaigns/1001/trigger.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "leads": [{ "id": 5678 }]
    }
  }'
```

## Reference Files

Detailed endpoint documentation by domain:

- [Identity & Authentication](references/identity.md) — OAuth setup, token lifecycle, permissions, rate limits
- [Leads & Companies](references/leads.md) — lead CRUD, activities, lead changes, custom activity types, companies, opportunities, field schemas, custom objects, custom object type schemas
- [Emails & Email Templates](references/emails.md) — email assets, content sections, modules, templates, transactional email send
- [Forms & Form Fields](references/forms.md) — form CRUD, field management, submit forms
- [Landing Pages](references/landing-pages.md) — landing pages, templates, redirect rules, content
- [Programs & Smart Campaigns](references/programs.md) — programs, smart campaigns, smart lists, tokens, tags, segments, snippets, folders, files
- [Bulk Import/Export](references/bulk-operations.md) — high-volume lead, activity, custom object, and program member operations
- [User Management](references/user-management.md) — users, roles, workspaces, invitations
