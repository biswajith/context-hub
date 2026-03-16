# Forms & Form Fields

Marketo Engage Asset API endpoints for managing forms, form fields, visibility rules, and programmatic form submission.

Base URL: `https://{munchkin-id}.mktorest.com`
All requests require: `-H "Authorization: Bearer $TOKEN"`

Asset POST endpoints use `application/x-www-form-urlencoded` unless noted. Folder parameters are passed as JSON strings, e.g. `{"id":123,"type":"Folder"}`.

---

## Forms

### Browse Forms

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/forms.json?maxReturn=20&offset=0&status=approved"
```

| Param | Type | Description |
|-------|------|-------------|
| status | string | `approved` or `draft`. Omit for both. |
| folder | JSON string | Filter by folder: `{"id":123,"type":"Folder"}` |
| maxReturn | integer | Max results per page (default 20, max 200) |
| offset | integer | Pagination offset |

Returns an array of form objects with `id`, `name`, `status`, `folder`, `url`, `language`, `locale`, `progressiveProfiling`, `createdAt`, and `updatedAt`.

### Get Form by Id

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001.json?status=draft"
```

Optional `status` param: `approved` or `draft`. Default returns approved version.

### Get Form by Name

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/byName.json?name=Contact%20Us"
```

| Param | Type | Description |
|-------|------|-------------|
| name | string | Required. Exact form name (URL-encode spaces). |
| status | string | Optional. `approved` or `draft`. |

### Create Form

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/forms.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Webinar Registration' \
  -d 'folder={"id":456,"type":"Folder"}' \
  -d 'description=Sign up for our upcoming webinar' \
  -d 'language=English' \
  -d 'locale=en_US' \
  -d 'progressiveProfiling=false' \
  -d 'labelWidth=100' \
  -d 'fieldWidth=300' \
  -d 'waitingLabel=Please wait...' \
  -d 'thankYouLabel=Thank you for registering!'
```

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Form name |
| folder | JSON string | Yes | Target folder: `{"id":456,"type":"Folder"}` |
| description | string | No | Form description |
| language | string | No | Form language (e.g. `English`) |
| locale | string | No | Locale code (e.g. `en_US`) |
| progressiveProfiling | boolean | No | Enable progressive profiling |
| labelWidth | integer | No | Label width in pixels |
| fieldWidth | integer | No | Field width in pixels |
| waitingLabel | string | No | Text shown while form submits |
| thankYouLabel | string | No | Thank-you page headline text |
| thankYouList | JSON string | No | Follow-up config (see below) |
| knownVisitor | JSON string | No | Known visitor behavior (see below) |

**thankYouList** — `followupType` can be `url`, `lp` (landing page ID), or `default`:

```json
{"followupType":"url","followupValue":"https://example.com/thank-you"}
```

**knownVisitor** — `type` can be `custom` with an HTML `template`:

```json
{"type":"custom","template":"<p>Welcome back, {{lead.FirstName}}!</p>"}
```

### Update Form Metadata

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Webinar Registration (Updated)' \
  -d 'description=Updated description' \
  -d 'progressiveProfiling=true'
```

Accepts the same optional parameters as Create Form. Only included fields are updated.

### Delete Form

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

The form must be unapproved and have no references from landing pages.

### Approve Form Draft

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/approveDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

Approves the current draft, making it the live approved version.

### Discard Form Approval

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/unapprove.json" \
  -H "Authorization: Bearer $TOKEN"
```

Reverts the form to draft-only state. The form will no longer render on landing pages.

### Discard Form Draft

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/discardDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

Discards the current draft, leaving the approved version unchanged.

### Clone Form

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Webinar Registration Copy' \
  -d 'folder={"id":456,"type":"Folder"}' \
  -d 'description=Cloned form'
```

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Name for the cloned form |
| folder | JSON string | Yes | Destination folder |
| description | string | No | Description for the clone |

---

## Form Fields

### Get Form Fields

Returns all fields configured on a form.

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/fields.json?status=approved"
```

Optional `status` param: `approved` or `draft`. Each result object includes `id`, `label`, `dataType`, `rowNumber`, `columnNumber`, `required`, and `fieldMetaData`.

### Add Form Field Set

Adds one or more fields to a form in a single call.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/fields.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'field=[{"fieldId":"Email","label":"Email Address","isRequired":true},{"fieldId":"FirstName","label":"First Name","isRequired":false},{"fieldId":"Company","label":"Company Name","isRequired":false}]'
```

The `field` parameter is a JSON array. Each entry supports:

| Property | Type | Description |
|----------|------|-------------|
| fieldId | string | Required. API name of the lead field. |
| label | string | Display label |
| isRequired | boolean | Whether the field is mandatory |
| labelWidth | integer | Label width in pixels |
| fieldWidth | integer | Field width in pixels |

### Update Form Field

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/fields/Email.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'label=Work Email' \
  -d 'fieldWidth=350' \
  -d 'instructions=Enter your work email address' \
  -d 'required=true' \
  -d 'hintText=you@company.com'
```

| Param | Type | Description |
|-------|------|-------------|
| label | string | Display label |
| fieldWidth | integer | Field width in pixels |
| labelWidth | integer | Label width in pixels |
| instructions | string | Help text displayed below the field |
| required | boolean | Whether the field is mandatory |
| hintText | string | Placeholder text inside the field |
| defaultValue | string | Pre-filled value |

### Delete Form Field

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/fields/Company/delete.json" \
  -H "Authorization: Bearer $TOKEN"
```

Removes a field from the form. Does not delete the underlying lead field.

### Rearrange Form Fields

Every field on the form must be included in the positions array.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/reArrange.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'positions=[{"columnNumber":0,"rowNumber":0,"fieldName":"Email"},{"columnNumber":0,"rowNumber":1,"fieldName":"FirstName"},{"columnNumber":0,"rowNumber":2,"fieldName":"LastName"},{"columnNumber":0,"rowNumber":3,"fieldName":"Company"}]'
```

| Property | Type | Description |
|----------|------|-------------|
| fieldName | string | Field API name |
| rowNumber | integer | Row position (0-based) |
| columnNumber | integer | Column position (0-based, typically 0) |

### Get Available Form Fields

Returns all lead fields available for use on forms (not specific to any form).

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/fields.json"
```

### Update Form Field Visibility Rules

Controls when a field is shown or hidden based on other field values.

```bash
curl -s -X POST \
  "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/1001/fields/Company/visibility.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'visibilityRule={"ruleType":"show","rules":[{"subjectField":"Email","operator":"isNotEmpty","values":[]}]}'
```

The `visibilityRule` parameter is a JSON object:

| Property | Type | Description |
|----------|------|-------------|
| ruleType | string | `show` or `hide` |
| rules[].subjectField | string | Field API name that drives visibility |
| rules[].operator | string | `is`, `isNot`, `isEmpty`, `isNotEmpty`, `contains`, `startsWith` |
| rules[].values | array | Values to match (empty array for `isEmpty`/`isNotEmpty`) |

Multiple rules are combined with AND logic.

---

## Submit Form

This endpoint lives under `/rest/v1/` (not the Asset API) and uses a **JSON request body**.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/v1/leads/submitForm.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "formId": 1001,
    "input": [
      {
        "leadFormFields": {
          "Email": "jane@example.com",
          "FirstName": "Jane",
          "LastName": "Smith",
          "Company": "Acme Corp"
        },
        "visitorData": {
          "pageURL": "https://www.example.com/contact",
          "queryString": "utm_source=google&utm_medium=cpc",
          "userAgentString": "Mozilla/5.0"
        },
        "cookie": "id:561-HYG-937&token:_mch-example.com-1234567890"
      }
    ]
  }'
```

| Field | Type | Description |
|-------|------|-------------|
| formId | integer | Required. Marketo form ID. |
| input[].leadFormFields | object | Required. Field API name to value map. |
| input[].visitorData | object | Optional. Contains `pageURL`, `queryString`, `userAgentString`. |
| input[].cookie | string | Optional. Munchkin tracking cookie for lead association. |

Response `result[].status` is `created` or `updated` depending on whether the lead already existed.

---

## Error Handling

Form-specific errors (in addition to standard Marketo codes):

| Code | Message | Cause |
|------|---------|-------|
| 702 | Record not found | Form ID does not exist |
| 709 | Folder not found | Specified folder does not exist |
| 710 | Name already in use | Duplicate name in the folder |
| 611 | System error | Retry with exponential backoff |

---

## Workflow Example

End-to-end: create a form, add fields, arrange, approve, and submit.

```bash
# 1. Create the form
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/forms.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'name=Demo Request' -d 'folder={"id":100,"type":"Folder"}'

# 2. Add fields (assume form ID 2001 returned)
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/2001/fields.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'field=[{"fieldId":"Email","label":"Email","isRequired":true},{"fieldId":"FirstName","label":"First Name"},{"fieldId":"LastName","label":"Last Name"},{"fieldId":"Company","label":"Company"}]'

# 3. Rearrange fields
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/2001/reArrange.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'positions=[{"columnNumber":0,"rowNumber":0,"fieldName":"FirstName"},{"columnNumber":0,"rowNumber":1,"fieldName":"LastName"},{"columnNumber":0,"rowNumber":2,"fieldName":"Email"},{"columnNumber":0,"rowNumber":3,"fieldName":"Company"}]'

# 4. Approve the form
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/asset/v1/form/2001/approveDraft.json" \
  -H "Authorization: Bearer $TOKEN"

# 5. Submit a lead through the form
curl -s -X POST "https://{munchkin-id}.mktorest.com/rest/v1/leads/submitForm.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"formId":2001,"input":[{"leadFormFields":{"Email":"demo@example.com","FirstName":"Alex","LastName":"Chen","Company":"TechCo"}}]}'
```
