# Emails & Email Templates

Marketo Engage REST API reference for emails, email content, modules, variables, and email templates.

Base URL: `https://{munchkin-id}.mktorest.com`
Auth header: `-H "Authorization: Bearer $TOKEN"`
POST bodies use `application/x-www-form-urlencoded` unless noted. File uploads use `multipart/form-data`. Responses are JSON.

---

## Emails

### Browse Emails

`GET /rest/asset/v1/emails.json`

| Param | Type | Description |
|-------|------|-------------|
| status | string | `approved` or `draft` |
| folder | JSON | `{"id":123,"type":"Folder"}` |
| maxReturn | int | Default 20, max 200 |
| offset | int | Paging offset |

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emails.json?maxReturn=2&status=approved" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "success": true,
  "result": [{
    "id": 1001, "name": "Welcome Email", "status": "approved",
    "folder": {"type": "Folder", "value": 42, "folderName": "Onboarding"},
    "subject": {"type": "Text", "value": "Welcome to Acme!"},
    "fromName": {"type": "Text", "value": "Acme Team"},
    "fromEmail": {"type": "Text", "value": "team@acme.com"},
    "createdAt": "2025-06-01T10:00:00Z+0000",
    "updatedAt": "2025-08-15T14:30:00Z+0000"
  }]
}
```

### Get Email by Id / by Name

- `GET /rest/asset/v1/email/{id}.json` — optional `status` param
- `GET /rest/asset/v1/email/byName.json` — required `name`, optional `status`, `folder`

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001.json?status=draft" \
  -H "Authorization: Bearer $TOKEN"
```

### Create Email

`POST /rest/asset/v1/emails.json`

| Param | Type | Required |
|-------|------|----------|
| name | string | yes |
| folder | JSON | yes |
| template | int | yes |
| subject | string | no |
| fromName | string | no |
| fromEmail | string | no |
| replyEmail | string | no |
| description | string | no |

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emails.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=July Newsletter' \
  -d 'folder={"id":42,"type":"Folder"}' \
  -d 'template=1050' \
  -d 'subject=Your July Update' \
  -d 'fromName=Acme Marketing' \
  -d 'fromEmail=marketing@acme.com'
```

```json
{
  "success": true,
  "result": [{
    "id": 1055, "name": "July Newsletter", "status": "draft",
    "template": 1050,
    "subject": {"type": "Text", "value": "Your July Update"},
    "fromName": {"type": "Text", "value": "Acme Marketing"}
  }]
}
```

### Update Email Metadata

`POST /rest/asset/v1/email/{id}.json`

Updates name, description, subject, fromName, fromEmail, replyEmail on the draft.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1055.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'subject=Updated: Your July Highlights&fromName=Acme Team'
```

### Delete Email

`POST /rest/asset/v1/email/{id}/delete.json` — email must be unapproved first.

```bash
curl -s -X POST "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1055/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Approve / Unapprove / Discard Draft

No body required — just POST with auth header.

| Action | Endpoint |
|--------|----------|
| Approve draft | `POST /rest/asset/v1/email/{id}/approveDraft.json` |
| Unapprove | `POST /rest/asset/v1/email/{id}/unapprove.json` |
| Discard draft | `POST /rest/asset/v1/email/{id}/discardDraft.json` |

```bash
curl -s -X POST "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1055/approveDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Clone Email

`POST /rest/asset/v1/email/{id}/clone.json`

| Param | Type | Required |
|-------|------|----------|
| name | string | yes |
| folder | JSON | yes |
| description | string | no |
| operational | boolean | no |

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Welcome Email v2&folder={"id":42,"type":"Folder"}'
```

### Send Sample Email

`POST /rest/asset/v1/email/{id}/sendSample.json`

| Param | Type | Required |
|-------|------|----------|
| emailAddress | string | yes |
| textOnly | boolean | no |
| leadId | int | no |

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/sendSample.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'emailAddress=test@acme.com&leadId=5042'
```

---

## Email Content

### Get Email Content

`GET /rest/asset/v1/email/{id}/content.json` — optional `status` param.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/content.json" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "success": true,
  "result": [
    {"htmlId": "hero_headline", "contentType": "Text", "value": "<h1>Welcome!</h1>"},
    {"htmlId": "body_copy", "contentType": "Text", "value": "<p>Thanks for signing up.</p>"}
  ]
}
```

### Update Email Content (Header Fields)

`POST /rest/asset/v1/email/{id}/content.json`

Updates subject, fromEmail, fromName, replyTo. Values are JSON-encoded type/value objects.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/content.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'subject={"type":"Text","value":"New Subject Line"}' \
  -d 'fromEmail={"type":"Text","value":"hello@acme.com"}'
```

### Update Email Content Section

`POST /rest/asset/v1/email/{id}/content/{htmlId}.json`

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| type | string | yes | `Text`, `DynamicContent`, or `Snippet` |
| value | string | conditional | HTML content or snippet/segmentation id |

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/content/hero_headline.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'type=Text&value=<h1>Welcome to Acme, {{lead.First Name}}!</h1>'
```

### Dynamic Content

- `GET /rest/asset/v1/email/{id}/dynamicContent/{contentId}.json` — get segmented variations
- `POST /rest/asset/v1/email/{id}/dynamicContent/{contentId}.json` — update a segment

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/dynamicContent/hero_headline.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'segment=Industry&value=<h1>Welcome, Tech Leader!</h1>&type=Text'
```

---

## Email Modules

Email 2.0 templates support modular sections that can be added, removed, duplicated, and reordered.

### Add Module

`POST /rest/asset/v1/email/{id}/content/{moduleId}/add.json` — params: `name` (required), `index` (required)

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/content/cta_module/add.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=cta_module_copy&index=3'
```

### Delete / Duplicate / Rename Module

| Action | Endpoint | Params |
|--------|----------|--------|
| Delete | `POST .../content/{moduleId}/delete.json` | none |
| Duplicate | `POST .../content/{moduleId}/duplicate.json` | `name` (required) |
| Rename | `POST .../content/{moduleId}/rename.json` | `name` (required) |

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/content/hero_module/duplicate.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=hero_module_v2'
```

### Rearrange Email Modules

`POST /rest/asset/v1/email/{id}/content/rearrange.json`

The `positions` param is a JSON array with `index` and `name` per module.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/content/rearrange.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'positions=[{"index":0,"name":"header_module"},{"index":1,"name":"hero_module"},{"index":2,"name":"cta_module"}]'
```

---

## Email Variables

Template-level variables (colors, toggles, text) defined via `mktoString`, `mktoBoolean`, `mktoColor`, etc.

### Get Email Variables

`GET /rest/asset/v1/email/{id}/variables.json`

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/variables.json" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "success": true,
  "result": [
    {"name": "brandColor", "type": "Color", "value": "#FF5733"},
    {"name": "showBanner", "type": "Boolean", "value": "true"},
    {"name": "heroTitle", "type": "String", "value": "Welcome!"}
  ]
}
```

### Update Email Variable

`POST /rest/asset/v1/email/{id}/variable/{name}.json`

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/1001/variable/brandColor.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'value=#2196F3'
```

---

## Email CC Fields

`GET /rest/asset/v1/email/ccFields.json` — returns fields available for Email CC.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/email/ccFields.json" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{"success": true, "result": [{"field": "ccField1", "label": "CC Field 1"}]}
```

---

## Email Templates

### Browse Email Templates

`GET /rest/asset/v1/emailTemplates.json`

Same paging params as Browse Emails: `status`, `folder`, `maxReturn`, `offset`.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplates.json?maxReturn=5&status=approved" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "success": true,
  "result": [{
    "id": 1050, "name": "Standard Newsletter", "status": "approved",
    "folder": {"type": "Folder", "value": 15, "folderName": "Email Templates"},
    "createdAt": "2025-01-10T08:00:00Z+0000",
    "updatedAt": "2025-03-22T12:00:00Z+0000"
  }]
}
```

### Get Email Template by Id / by Name

- `GET /rest/asset/v1/emailTemplate/{id}.json`
- `GET /rest/asset/v1/emailTemplate/byName.json?name={name}`

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/byName.json?name=Standard%20Newsletter" \
  -H "Authorization: Bearer $TOKEN"
```

### Create Email Template

`POST /rest/asset/v1/emailTemplates.json`

Provide HTML via `content` param or upload a file via `multipart/form-data`.

| Param | Type | Required |
|-------|------|----------|
| name | string | yes |
| folder | JSON | yes |
| content | string | conditional |
| description | string | no |

**Inline content:**

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplates.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Promo Template' \
  -d 'folder={"id":15,"type":"Folder"}' \
  --data-urlencode 'content=<html><body><div class="mktoEditable" id="main">Edit here</div></body></html>'
```

**File upload:**

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplates.json" \
  -H "Authorization: Bearer $TOKEN" \
  -F "name=Promo Template" \
  -F 'folder={"id":15,"type":"Folder"}' \
  -F "content=@template.html"
```

### Update Email Template Metadata

`POST /rest/asset/v1/emailTemplate/{id}.json`

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/1050.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Standard Newsletter v2&description=Updated layout'
```

### Delete Email Template

`POST /rest/asset/v1/emailTemplate/{id}/delete.json` — must not be in use by any emails.

```bash
curl -s -X POST "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/1050/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Approve / Unapprove / Discard Draft

| Action | Endpoint |
|--------|----------|
| Approve draft | `POST /rest/asset/v1/emailTemplate/{id}/approveDraft.json` |
| Unapprove | `POST /rest/asset/v1/emailTemplate/{id}/unapprove.json` |
| Discard draft | `POST /rest/asset/v1/emailTemplate/{id}/discardDraft.json` |

Cannot unapprove a template in use by approved emails.

```bash
curl -s -X POST "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/1050/approveDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Clone Email Template

`POST /rest/asset/v1/emailTemplate/{id}/clone.json`

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/1050/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Standard Newsletter Copy&folder={"id":15,"type":"Folder"}'
```

### Get Email Template Content

`GET /rest/asset/v1/emailTemplate/{id}/content` — no `.json` extension. Optional `?status=draft`.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/1050/content?status=approved" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{"success": true, "result": [{"content": "<html>...template HTML...</html>", "contentType": "HTML"}]}
```

### Update Email Template Content

`POST /rest/asset/v1/emailTemplate/{id}/content.json` — file upload or form-encoded `content`.

```bash
curl -s "https://123-ABC-456.mktorest.com/rest/asset/v1/emailTemplate/1050/content.json" \
  -H "Authorization: Bearer $TOKEN" \
  -F "content=@updated-template.html"
```

---

## Common Response Patterns

**Success:** `{"success": true, "result": [{...}]}`

**Error:** `{"success": false, "errors": [{"code": "702", "message": "Email not found"}]}`

| Code | Meaning |
|------|---------|
| 601 | Access token invalid |
| 602 | Access token expired |
| 702 | Record not found |
| 709 | Folder not found |
| 611 | Asset name already in use |

---

## Notes

- All write operations create or modify the **draft** version. Use `approveDraft` to publish.
- Approved assets can have both an approved and a draft version simultaneously.
- The `folder` param in create/clone endpoints is JSON-encoded: `{"id":123,"type":"Folder"}` or `{"id":456,"type":"Program"}`.
- Browse endpoints page via `offset` and `maxReturn` (default 20, max 200).
- Content sections are identified by `htmlId` values from the template markup (`class="mktoEditable"` with an `id` attribute).
