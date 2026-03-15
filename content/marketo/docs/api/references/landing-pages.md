# Landing Pages

Marketo Asset API endpoints for managing landing pages, landing page content sections, landing page templates, and redirect rules.

Base URL: `https://{munchkin-id}.mktorest.com`
All requests require: `-H "Authorization: Bearer $TOKEN"`

Asset POST endpoints use `application/x-www-form-urlencoded` unless noted. Folder parameters accept JSON strings: `{"id":123,"type":"Folder"}`.

## Browse Landing Pages

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPages.json?maxReturn=20&offset=0&status=approved"
```

Query params: `maxReturn` (max 200, default 20), `offset`, `status` (approved|draft), `folder` (JSON: `{"id":123,"type":"Folder"}`).

## Get Landing Page by Id

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001.json?status=draft"
```

Query param `status`: `approved` or `draft`. Omit to get the approved version (or draft if no approved version exists).

## Get Landing Page by Name

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/byName.json?name=My%20Landing%20Page"
```

## Create Landing Page

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPages.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=My New Landing Page' \
  -d 'folder={"id":123,"type":"Folder"}' \
  -d 'template=45' \
  -d 'description=Promo page for Q3 campaign' \
  -d 'title=Special Offer' \
  -d 'keywords=promo,offer,q3' \
  -d 'robots=index,follow' \
  -d 'mobileEnabled=true' \
  -d 'prefillForm=false'
```

Required: `name`, `folder` (JSON), `template` (integer template ID). Optional: `description`, `title`, `keywords`, `robots`, `customHeadHTML`, `facebookOgTags`, `prefillForm`, `mobileEnabled`, `URL`.

## Update Landing Page Metadata

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'description=Updated description' \
  -d 'title=New Page Title' \
  -d 'keywords=updated,keywords'
```

Accepts the same optional fields as create. Changes apply to the draft version.

## Delete Landing Page

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

Cannot delete an approved landing page — unapprove it first.

## Approve Landing Page Draft

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/approveDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

## Unapprove Landing Page

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/unapprove.json" \
  -H "Authorization: Bearer $TOKEN"
```

Removes the page from the live site. Fails if the page is referenced by other assets.

## Discard Landing Page Draft

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/discardDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

Reverts the draft to the current approved version.

## Clone Landing Page

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Cloned Landing Page' \
  -d 'folder={"id":456,"type":"Folder"}'
```

Required: `name`, `folder` (JSON).

---

## Landing Page Content

### Get Landing Page Content

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/content.json?status=draft"
```

Returns all content sections for the page. Each section has an `id`, `type` (RichText, HTML, Form, DynamicContent, Rectangle, Snippet), and `content` value.

### Update Landing Page Content Section

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/content.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'type=RichText' \
  -d 'id=section01' \
  -d 'value=<h1>Welcome</h1><p>Check out our latest offer.</p>'
```

The `id` must match an existing editable section from the template. For `Form` type, `value` is the form ID. For `DynamicContent`, `value` is the segmentation ID.

### Delete Landing Page Content Section

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPage/1001/content/section01/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Landing Page Templates

### Browse Templates

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplates.json?maxReturn=20&offset=0&status=approved"
```

Query params: `maxReturn`, `offset`, `status` (approved|draft), `folder` (JSON).

### Get Template by Id

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/50.json?status=approved"
```

### Get Template by Name

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/byName.json?name=My%20Template"
```

### Create Template

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplates.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=New LP Template' \
  -d 'folder={"id":123,"type":"Folder"}' \
  -d 'description=Template for promo pages'
```

Required: `name`, `folder` (JSON). Template content is set separately via the content endpoint.

### Update Template

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/50.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Renamed Template' \
  -d 'description=Updated description'
```

### Delete Template

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/50/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

Cannot delete a template that is in use by landing pages.

### Approve Template

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/50/approveDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Template Content

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/50/content.json"
```

Returns the raw HTML of the template. Editable sections are marked with `mktoText`, `mktoImg`, `mktoForm`, etc. class attributes.

### Update Template Content

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/landingPageTemplate/50/content.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'content=<html><head><title>LP</title></head><body><div class="mktoText" id="section01">Default</div></body></html>'
```

The HTML must include at least one Marketo editable element (`mktoText`, `mktoImg`, `mktoForm`, `mktoSnippet`).

---

## Landing Page Redirect Rules

### Get Redirect Rules

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/redirectRules.json?maxReturn=20&offset=0"
```

Query params: `maxReturn`, `offset`.

### Get Redirect Rule by Id

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/redirectRule/100.json"
```

### Create Redirect Rule

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/redirectRules.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'hostname=pages.example.com' \
  -d 'redirectFrom=/old-path' \
  -d 'redirectTo=/new-path'
```

Required: `hostname`, `redirectFrom`, `redirectTo`. The `redirectTo` can be a Marketo landing page path or an external URL.

### Update Redirect Rule

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/redirectRule/100.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'redirectTo=/updated-path'
```

### Delete Redirect Rule

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/redirectRule/100/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Response Format

All endpoints return the standard Marketo envelope:

```json
{
  "requestId": "a1b2#18a1f2c3d4e",
  "success": true,
  "result": [ { "id": 1001, "name": "My Landing Page", "status": "approved", "url": "/lp/my-landing-page" } ]
}
```

On error, `success` is `false` and an `errors` array is returned. Common asset error codes: `702` (record not found), `709` (access violation), `710` (parent folder not found), `611` (system error).
