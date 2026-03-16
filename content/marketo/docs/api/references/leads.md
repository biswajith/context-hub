# Leads & Companies

Marketo Engage REST API reference for managing leads, companies, opportunities, activities, lists, custom objects, and related entities.

**Base URL:** `https://{munchkin-id}.mktorest.com`
**Auth:** All requests require `-H "Authorization: Bearer $TOKEN"`
**Content-Type:** `application/json` for all POST/PUT bodies. Responses are JSON.

---

## Leads

Core entity in Marketo. Leads represent people (prospects, customers, contacts).

### Get Leads by Filter Type

```
GET /rest/v1/leads.json?filterType={field}&filterValues={csv}&fields={csv}&batchSize={int}&nextPageToken={token}
```

- `filterType` (required) — field to filter on (e.g. `email`, `id`, `company`)
- `filterValues` (required) — comma-separated values
- `fields` — comma-separated list of fields to return
- `batchSize` — max 300 (default 300)
- `nextPageToken` — for paging through results

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/leads.json?filterType=email&filterValues=jdoe@example.com,asmith@example.com&fields=id,email,firstName,lastName,company" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "1234#abc",
  "result": [
    { "id": 42, "email": "jdoe@example.com", "firstName": "John", "lastName": "Doe", "company": "Acme" }
  ],
  "success": true
}
```

### Get Lead by Id

```
GET /rest/v1/lead/{id}.json?fields={csv}
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/lead/42.json?fields=email,firstName,lastName" \
  -H "Authorization: Bearer $TOKEN"
```

### Create/Update Leads

```
POST /rest/v1/leads.json
```

Body parameters:
- `action` — `createOnly`, `updateOnly`, `createOrUpdate` (default), or `createDuplicate`
- `lookupField` — field used for deduplication (default `email`)
- `input` — array of lead objects

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/leads.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createOrUpdate",
    "lookupField": "email",
    "input": [
      { "email": "jdoe@example.com", "firstName": "John", "lastName": "Doe", "company": "Acme" }
    ]
  }'
```

```json
{
  "requestId": "5678#def",
  "result": [{ "id": 42, "status": "updated" }],
  "success": true
}
```

### Delete Leads

```
POST /rest/v1/leads/delete.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/leads/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "input": [{ "id": 42 }, { "id": 43 }] }'
```

### Describe Lead

Returns field metadata for the lead object.

```
GET /rest/v1/leads/describe.json
GET /rest/v1/leads/describe2.json  (more detail, includes searchable fields and custom objects)
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/leads/describe2.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Merge Leads

Merges one or more leads into a winning lead. The losing leads are deleted.

```
POST /rest/v1/leads/{id}/merge.json?leadIds={csv}&mergeInCRM={bool}
```

- `{id}` — the winning lead id (path param)
- `leadIds` — comma-separated losing lead ids
- `mergeInCRM` — if `true`, also merges in the synced CRM

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/leads/42/merge.json?leadIds=43,44&mergeInCRM=false" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Lead Partitions

```
GET /rest/v1/lead/partitions.json
```

Returns available lead partitions (relevant for multi-partition instances).

---

## Activities

Track lead interactions and behavior changes.

### Get Paging Token

Required before fetching activities. Returns a token representing a point in time.

```
GET /rest/v1/activities/pagingtoken.json?sinceDatetime={ISO8601}
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities/pagingtoken.json?sinceDatetime=2025-01-01T00:00:00Z" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{ "requestId": "abc#123", "success": true, "nextPageToken": "WQV2VQVPPCKHC..." }
```

### Get Lead Activities

```
GET /rest/v1/activities.json?activityTypeIds={csv}&nextPageToken={token}&batchSize={int}&leadIds={csv}&listId={int}
```

- `nextPageToken` (required) — from Get Paging Token or previous page
- `activityTypeIds` (required) — comma-separated activity type ids
- `batchSize` — max 300
- `leadIds` / `listId` — optional filters

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities.json?activityTypeIds=1,12&nextPageToken=WQV2VQVPPCKHC&batchSize=100" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "789#ghi",
  "success": true,
  "moreResult": true,
  "nextPageToken": "NEWTOKEN...",
  "result": [
    { "id": 101, "leadId": 42, "activityDate": "2025-06-15T10:30:00Z", "activityTypeId": 1, "primaryAttributeValue": "Visited Page" }
  ]
}
```

### Get Activity Types

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities/types.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Lead Changes

Returns data value changes and new lead activities for a set of leads.

```
GET /rest/v1/activities/leadchanges.json?nextPageToken={token}&fields={csv}&batchSize={int}&listId={int}
```

- `nextPageToken` (required) — from Get Paging Token
- `fields` (required) — comma-separated list of fields to track changes for

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities/leadchanges.json?\
nextPageToken=WQV2VQVPPCKHC&fields=email,firstName,lastName,company&batchSize=100" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "abc#123",
  "success": true,
  "moreResult": false,
  "result": [{
    "id": 5001,
    "leadId": 42,
    "activityDate": "2025-06-15T10:30:00Z",
    "activityTypeId": 13,
    "fields": [
      { "id": 6, "name": "company", "newValue": "Acme Inc", "oldValue": "Acme Corp" }
    ]
  }]
}
```

### Get Deleted Leads

Returns leads that were deleted after a given datetime.

```
GET /rest/v1/activities/deletedleads.json?nextPageToken={token}&batchSize={int}
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities/deletedleads.json?\
nextPageToken=WQV2VQVPPCKHC&batchSize=100" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "abc#456",
  "success": true,
  "result": [
    { "id": 99, "marketoGUID": "abc-def-ghi", "leadId": 99, "activityDate": "2025-06-10T08:00:00Z", "activityTypeId": 37 }
  ]
}
```

### Add Custom Activities

```
POST /rest/v1/activities/external.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/activities/external.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [{
      "leadId": 42,
      "activityDate": "2025-06-15T10:30:00Z",
      "activityTypeId": 100001,
      "primaryAttributeValue": "Trial Started",
      "attributes": [{ "name": "plan", "value": "Pro" }]
    }]
  }'
```

### Custom Activity Types

Manage custom activity type definitions. Requires **Read-Write Activity Metadata** permission.

**Create Custom Activity Type:**

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/activities/external/type.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "apiName": "myCustomActivity",
    "name": "My Custom Activity",
    "description": "Tracks custom user events",
    "primaryAttribute": {
      "name": "Event Name",
      "apiName": "eventName",
      "dataType": "string"
    }
  }'
```

**Get Custom Activity Types:**

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities/external/types.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Describe Custom Activity Type:**

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/activities/external/type/myCustomActivity/describe.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Add Attributes to Custom Activity Type:**

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/activities/external/type/myCustomActivity/attributes/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "attributes": [
      { "name": "Plan Name", "apiName": "planName", "dataType": "string" },
      { "name": "Amount", "apiName": "amount", "dataType": "integer" }
    ]
  }'
```

**Approve Custom Activity Type:**

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/activities/external/type/myCustomActivity/approve.json" \
  -H "Authorization: Bearer $TOKEN"
```

Other operations: `POST .../update.json` (update type), `POST .../attributes/update.json`, `POST .../attributes/delete.json`, `POST .../delete.json`, `POST .../discardDraft.json`.

---

## Lead Field Schema

Manage lead field definitions. Requires **Read-Write Schema Custom Field** permission.

### Get Lead Fields

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/leads/schema/fields.json?batchSize=300" \
  -H "Authorization: Bearer $TOKEN"
```

Returns all lead fields with `id`, `displayName`, `dataType`, `rest.name` (API name), `rest.readOnly`, `soap.name`, and `soap.readOnly`.

### Get Lead Field by Name

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/leads/schema/fields/company.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Create Lead Field

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/leads/schema/fields.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [{
      "displayName": "Favorite Color",
      "dataType": "string",
      "name": "favoriteColor",
      "description": "Lead preferred color"
    }]
  }'
```

### Update Lead Field

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/leads/schema/fields/favoriteColor.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [{
      "displayName": "Preferred Color",
      "description": "Updated description"
    }]
  }'
```

### Company Fields

```bash
# List all company fields
curl "https://123-ABC-456.mktorest.com/rest/v1/companies/schema/fields.json" \
  -H "Authorization: Bearer $TOKEN"

# Get specific company field
curl "https://123-ABC-456.mktorest.com/rest/v1/companies/schema/fields/annualRevenue.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Opportunity Fields

```bash
# List all opportunity fields
curl "https://123-ABC-456.mktorest.com/rest/v1/opportunities/schema/fields.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Companies

Company records that can be linked to leads.

### Get Companies

```
GET /rest/v1/companies.json?filterType={field}&filterValues={csv}
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/companies.json?filterType=company&filterValues=Acme%20Corp" \
  -H "Authorization: Bearer $TOKEN"
```

### Sync Companies

```
POST /rest/v1/companies.json
```

- `action` — `createOnly`, `updateOnly`, `createOrUpdate` (default)
- `dedupeBy` — `dedupeFields` (default) or `idField`
- `input` — array of company objects

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/companies.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createOrUpdate",
    "dedupeBy": "dedupeFields",
    "input": [{ "externalCompanyId": "C-001", "company": "Acme Corp", "website": "https://acme.com" }]
  }'
```

### Delete Companies

```
POST /rest/v1/companies/delete.json
```

### Describe Company

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/companies/describe.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Opportunities

Opportunity records and their role associations with leads.

### Get/Sync/Delete Opportunities

```
GET  /rest/v1/opportunities.json?filterType={field}&filterValues={csv}
POST /rest/v1/opportunities.json         — Sync (action, dedupeBy, input)
POST /rest/v1/opportunities/delete.json  — Delete
GET  /rest/v1/opportunities/describe.json
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/opportunities.json?filterType=externalOpportunityId&filterValues=OPP-001" \
  -H "Authorization: Bearer $TOKEN"
```

### Opportunity Roles

Link leads to opportunities via roles.

```
GET  /rest/v1/opportunities/roles.json?filterType={field}&filterValues={csv}
POST /rest/v1/opportunities/roles.json   — Sync roles (action, dedupeBy, input)
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/opportunities/roles.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createOrUpdate",
    "input": [{ "externalOpportunityId": "OPP-001", "leadId": 42, "role": "Decision Maker" }]
  }'
```

---

## Static Lists

Manually-managed collections of leads.

### Get Lists

```
GET /rest/v1/lists.json?id={csv}&name={csv}&programName={csv}&workspaceName={csv}
GET /rest/v1/lists/{listId}.json
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/lists.json?name=My%20Target%20List" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Leads by List Id

```
GET /rest/v1/lists/{listId}/leads.json?fields={csv}&batchSize={int}&nextPageToken={token}
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/lists/1001/leads.json?fields=email,firstName" \
  -H "Authorization: Bearer $TOKEN"
```

### Add Leads to List

```
POST /rest/v1/lists/{listId}/leads.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/lists/1001/leads.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "input": [{ "id": 42 }, { "id": 43 }] }'
```

### Remove Leads from List

```
DELETE /rest/v1/lists/{listId}/leads.json?id=42&id=43
```

### Member of List

```
GET /rest/v1/lists/{listId}/leads/ismember.json?id=42&id=43
```

```json
{
  "requestId": "abc#1",
  "result": [{ "id": 42, "status": "memberof" }, { "id": 43, "status": "notmemberof" }],
  "success": true
}
```

---

## Named Accounts (ABM)

Account-based marketing entities for targeted account strategies.

### Get/Sync/Delete Named Accounts

```
GET  /rest/v1/namedaccounts.json?filterType={field}&filterValues={csv}
POST /rest/v1/namedaccounts.json          — Sync (action, dedupeBy, input)
POST /rest/v1/namedaccounts/delete.json   — Delete
GET  /rest/v1/namedaccounts/describe.json — Describe
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/namedaccounts.json?filterType=name&filterValues=Acme%20Corp" \
  -H "Authorization: Bearer $TOKEN"
```

### Named Account Lists

```
GET    /rest/v1/namedAccountLists.json
POST   /rest/v1/namedAccountList/{id}/namedAccounts.json   — Add accounts to list
DELETE /rest/v1/namedAccountList/{id}/namedAccounts.json   — Remove accounts from list
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/namedAccountList/5/namedAccounts.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "input": [{ "name": "Acme Corp" }] }'
```

---

## Custom Objects

User-defined objects that extend the Marketo data model.

### List Custom Objects

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/customobjects.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get / Sync / Delete Custom Objects

```
GET  /rest/v1/customobjects/{name}.json?filterType={field}&filterValues={csv}
POST /rest/v1/customobjects/{name}.json          — Sync (action, dedupeBy, input)
POST /rest/v1/customobjects/{name}/delete.json   — Delete
GET  /rest/v1/customobjects/{name}/describe.json — Describe
```

`{name}` is the API name of the custom object (e.g. `car_c`).

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/customobjects/car_c.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createOrUpdate",
    "dedupeBy": "dedupeFields",
    "input": [{ "leadId": 42, "vin": "1HGCM82633A004352", "make": "Honda", "model": "Accord" }]
  }'
```

### Custom Object Type Schema

Manage custom object type definitions (create/modify the object schema itself). Requires **Read-Write Custom Object Type** permission.

```bash
# List custom object types
curl "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema.json" \
  -H "Authorization: Bearer $TOKEN"

# Describe a custom object type
curl "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/car_c/describe.json" \
  -H "Authorization: Bearer $TOKEN"

# Get linkable objects (for relationship fields)
curl "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/linkableObjects.json" \
  -H "Authorization: Bearer $TOKEN"

# Get available field data types
curl "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/fieldDataTypes.json" \
  -H "Authorization: Bearer $TOKEN"

# Get dependent assets for a custom object type
curl "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/car_c/dependentAssets.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Create Custom Object Type:**

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "createOnly",
    "displayName": "Car",
    "apiName": "car_c",
    "description": "Tracks cars owned by leads",
    "pluralName": "Cars",
    "showInLeadDetail": true
  }'
```

**Add Fields / Approve / Delete / Discard Draft:**

```bash
# Add fields
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/car_c/addField.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [
      { "displayName": "VIN", "name": "vin", "dataType": "string", "isDedupeField": true },
      { "displayName": "Make", "name": "make", "dataType": "string" }
    ]
  }'

# Approve (required before syncing data)
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/car_c/approve.json" \
  -H "Authorization: Bearer $TOKEN"

# Delete type
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/car_c/delete.json" \
  -H "Authorization: Bearer $TOKEN"

# Discard draft changes
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/customobjects/schema/car_c/discardDraft.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Sales Persons

Sales rep records synced from CRM or managed directly.

```
GET  /rest/v1/salespersons.json?filterType={field}&filterValues={csv}
POST /rest/v1/salespersons.json          — Sync (action, dedupeBy, input)
POST /rest/v1/salespersons/delete.json   — Delete
GET  /rest/v1/salespersons/describe.json — Describe
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/salespersons.json?filterType=email&filterValues=srep@example.com" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Program Members

Manage membership and status within Marketo programs (events, nurtures, etc.).

### Describe Program Member

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/programs/members/describe.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Program Members

```
GET /rest/v1/programs/{programId}/members.json?filterType=statusName&filterValues={csv}&fields={csv}&batchSize={int}&nextPageToken={token}
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/programs/1010/members.json?filterType=statusName&filterValues=Registered,Attended&fields=email,firstName,status" \
  -H "Authorization: Bearer $TOKEN"
```

### Sync Program Member Status

```
POST /rest/v1/programs/{programId}/members/status.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/rest/v1/programs/1010/members/status.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "statusName": "Attended",
    "input": [{ "leadId": 42 }, { "leadId": 43 }]
  }'
```

---

## Usage & Errors

Monitor API consumption and errors against daily quotas. Marketo enforces a 50,000 calls/day limit and a 10-second concurrent request limit per user.

```
GET /rest/v1/stats/usage.json              — Daily usage
GET /rest/v1/stats/errors.json             — Daily errors
GET /rest/v1/stats/usage/last7days.json    — Usage over last 7 days
GET /rest/v1/stats/errors/last7days.json   — Errors over last 7 days
```

```bash
curl "https://123-ABC-456.mktorest.com/rest/v1/stats/usage/last7days.json" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "abc#1",
  "success": true,
  "result": [
    { "date": "2025-06-14", "total": 4521 },
    { "date": "2025-06-13", "total": 3890 }
  ]
}
```

---

## Common Response Structure

All endpoints return this envelope:

```json
{
  "requestId": "{id}#{sequence}",
  "success": true,
  "result": [],
  "errors": [],
  "warnings": [],
  "nextPageToken": "...",
  "moreResult": false
}
```

- `success` — `false` indicates a request-level error (auth failure, bad endpoint)
- `errors` — array of `{ "code": "...", "message": "..." }` when `success` is `false`
- Per-record errors appear inside each `result` item as `status: "skipped"` with a `reasons` array
- `moreResult` / `nextPageToken` — present when results are paginated

## Common Error Codes

| Code | Meaning |
|------|---------|
| 601  | Access token invalid |
| 602  | Access token expired |
| 606  | Max rate limit exceeded |
| 607  | Daily quota reached |
| 610  | Requested resource not found |
| 611  | System error |
| 1001 | Invalid value — field-level validation failure |
| 1003 | Action not supported for the given entity |
| 1004 | Lead not found (for update/delete) |
| 1007 | Duplicate leads (when action=createOnly) |
