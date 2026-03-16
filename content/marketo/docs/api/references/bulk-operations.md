# Bulk Import/Export

Marketo Engage REST API reference for high-volume data transfer via Bulk Import and Export operations.

**Base URL:** `https://{munchkin-id}.mktorest.com`
**Auth:** All requests require `-H "Authorization: Bearer $TOKEN"`

---

## Overview

The Bulk API is designed for large data volumes where the standard REST endpoints are impractical.

- **Export** — asynchronous jobs that produce CSV/TSV/SSV files for download
- **Import** — accepts CSV/TSV/SSV file uploads via multipart/form-data

**Constraints:**
- Max 10 export jobs running simultaneously
- Max 10 import jobs simultaneously
- Export files available for 30 days after job completion
- Max import file size: 10MB
- Date filters (`createdAt`, `updatedAt`) require `startAt` and `endAt` in ISO 8601 format (e.g. `2025-01-01T00:00:00Z`)

**Job status lifecycle:** `Created` → `Queued` → `Processing` → `Completed` | `Failed` | `Cancelled`

---

## Bulk Export Leads

Workflow: **Create Job → Enqueue → Poll Status → Download File**

### Create Export Lead Job

```
POST /bulk/v1/leads/export/create.json
```

Body parameters:
- `fields` (required) — array of lead field names to include in export
- `format` — `CSV` (default), `TSV`, or `SSV`
- `columnHeaderNames` — object mapping field API names to custom header names
- `filter` — object with one of these time-based filters:
  - `createdAt` — `{ "startAt": "...", "endAt": "..." }`
  - `updatedAt` — `{ "startAt": "...", "endAt": "..." }`
  - `staticListName` / `staticListId` — filter by list membership
  - `smartListName` / `smartListId` — filter by smart list

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": ["id", "email", "firstName", "lastName", "company", "createdAt"],
    "format": "CSV",
    "columnHeaderNames": { "id": "Lead ID", "email": "Email Address" },
    "filter": {
      "createdAt": {
        "startAt": "2025-01-01T00:00:00Z",
        "endAt": "2025-06-30T23:59:59Z"
      }
    }
  }'
```

```json
{
  "requestId": "e42b#1234",
  "success": true,
  "result": [{
    "exportId": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
    "format": "CSV",
    "status": "Created",
    "createdAt": "2025-07-01T10:00:00Z"
  }]
}
```

### Enqueue Export Lead Job

```
POST /bulk/v1/leads/export/{exportId}/enqueue.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/a1b2c3d4-5678-90ab-cdef-1234567890ab/enqueue.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Export Lead Job Status

```
GET /bulk/v1/leads/export/{exportId}/status.json
```

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/a1b2c3d4-5678-90ab-cdef-1234567890ab/status.json" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "f1a2#5678",
  "success": true,
  "result": [{
    "exportId": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
    "format": "CSV",
    "status": "Completed",
    "fileSize": 1048576,
    "numberOfRecords": 5000,
    "createdAt": "2025-07-01T10:00:00Z",
    "finishedAt": "2025-07-01T10:05:00Z"
  }]
}
```

### Get Export Lead File

```
GET /bulk/v1/leads/export/{exportId}/file.json
```

Returns the raw CSV file content. Supports `Range` header for partial retrieval of large files.

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/a1b2c3d4-5678-90ab-cdef-1234567890ab/file.json" \
  -H "Authorization: Bearer $TOKEN" \
  -o leads_export.csv
```

Partial retrieval (first 500 bytes):

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/a1b2c3d4-5678-90ab-cdef-1234567890ab/file.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Range: bytes=0-499" \
  -o leads_partial.csv
```

### Cancel Export Lead Job

```
POST /bulk/v1/leads/export/{exportId}/cancel.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/a1b2c3d4-5678-90ab-cdef-1234567890ab/cancel.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Export Lead Jobs

List export jobs, optionally filtered by status.

```
GET /bulk/v1/leads/export.json?status={status}
```

- `status` — `Created`, `Queued`, `Processing`, `Completed`, `Failed`, `Cancelled`

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/export.json?status=Completed" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Bulk Export Activities

Same workflow pattern as leads: **Create → Enqueue → Poll → Download**.

### Create Export Activity Job

```
POST /bulk/v1/activities/export/create.json
```

Body parameters:
- `fields` — array of activity fields (e.g. `activityDate`, `activityTypeId`, `leadId`, `primaryAttributeValue`)
- `filter` (required):
  - `createdAt` — `{ "startAt": "...", "endAt": "..." }` (required)
  - `activityTypeIds` — array of integer activity type ids (optional, filters to specific types)
- `format` — `CSV`, `TSV`, or `SSV`

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/activities/export/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": ["leadId", "activityDate", "activityTypeId", "primaryAttributeValue"],
    "format": "CSV",
    "filter": {
      "createdAt": {
        "startAt": "2025-06-01T00:00:00Z",
        "endAt": "2025-06-30T23:59:59Z"
      },
      "activityTypeIds": [1, 12, 13]
    }
  }'
```

### Remaining Endpoints

```
POST /bulk/v1/activities/export/{exportId}/enqueue.json  — Enqueue
GET  /bulk/v1/activities/export/{exportId}/status.json   — Get Status
GET  /bulk/v1/activities/export/{exportId}/file.json     — Get File
POST /bulk/v1/activities/export/{exportId}/cancel.json   — Cancel
GET  /bulk/v1/activities/export.json                     — List Jobs (status filter)
```

Usage follows the same pattern as Bulk Export Leads — substitute the export ID returned from create.

---

## Bulk Export Custom Objects

### Create Export Custom Object Job

```
POST /bulk/v1/customobjects/{apiName}/export/create.json
```

`{apiName}` is the custom object API name (e.g. `car_c`).

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/customobjects/car_c/export/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": ["marketoGUID", "leadId", "vin", "make", "model"],
    "format": "CSV",
    "filter": {
      "createdAt": {
        "startAt": "2025-01-01T00:00:00Z",
        "endAt": "2025-06-30T23:59:59Z"
      }
    }
  }'
```

### Remaining Endpoints

```
POST /bulk/v1/customobjects/{apiName}/export/{exportId}/enqueue.json  — Enqueue
GET  /bulk/v1/customobjects/{apiName}/export/{exportId}/status.json   — Get Status
GET  /bulk/v1/customobjects/{apiName}/export/{exportId}/file.json     — Get File
POST /bulk/v1/customobjects/{apiName}/export/{exportId}/cancel.json   — Cancel
```

---

## Bulk Export Program Members

### Create Export Program Member Job

```
POST /bulk/v1/program/members/export/create.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/program/members/export/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": ["leadId", "email", "firstName", "lastName", "statusName", "reachedSuccess"],
    "format": "CSV",
    "filter": {
      "programId": 1010
    }
  }'
```

### Remaining Endpoints

```
POST /bulk/v1/program/members/export/{exportId}/enqueue.json  — Enqueue
GET  /bulk/v1/program/members/export/{exportId}/status.json   — Get Status
GET  /bulk/v1/program/members/export/{exportId}/file.json     — Get File
```

---

## Bulk Import Leads

Imports accept CSV/TSV/SSV files via `multipart/form-data`.

### Import Leads

```
POST /bulk/v1/leads.json
```

Form parameters:
- `file` (required) — the CSV/TSV/SSV file
- `format` (required) — `csv`, `tsv`, or `ssv`
- `lookupField` — field used for dedup (default `email`)
- `partitionName` — target lead partition (for multi-partition instances)
- `listId` — static list id to add imported leads to

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/leads.json" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@leads_to_import.csv" \
  -F "format=csv" \
  -F "lookupField=email" \
  -F "listId=1001"
```

```json
{
  "requestId": "d01f#1a2b",
  "success": true,
  "result": [{
    "batchId": 3456,
    "importId": "3456",
    "status": "Queued"
  }]
}
```

### Get Import Lead Status

```
GET /bulk/v1/leads/batch/{batchId}.json
```

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/batch/3456.json" \
  -H "Authorization: Bearer $TOKEN"
```

```json
{
  "requestId": "e02g#3c4d",
  "success": true,
  "result": [{
    "batchId": 3456,
    "status": "Complete",
    "numOfLeadsProcessed": 1500,
    "numOfRowsFailed": 3,
    "numOfRowsWithWarning": 12,
    "message": "Import completed with errors, please download failures and warnings."
  }]
}
```

### Get Import Lead Failures

```
GET /bulk/v1/leads/batch/{batchId}/failures.json
```

Returns CSV with failed rows and the reason for each failure.

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/batch/3456/failures.json" \
  -H "Authorization: Bearer $TOKEN" \
  -o import_failures.csv
```

### Get Import Lead Warnings

```
GET /bulk/v1/leads/batch/{batchId}/warnings.json
```

Returns CSV with rows that imported successfully but triggered warnings (e.g. duplicate leads merged).

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/batch/3456/warnings.json" \
  -H "Authorization: Bearer $TOKEN" \
  -o import_warnings.csv
```

---

## Bulk Import Custom Objects

### Import Custom Objects

```
POST /bulk/v1/customobjects/{apiName}/import.json
```

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/customobjects/car_c/import.json" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@custom_objects.csv" \
  -F "format=csv"
```

### Status, Failures, Warnings

```
GET /bulk/v1/customobjects/{apiName}/import/{batchId}/status.json    — Get Status
GET /bulk/v1/customobjects/{apiName}/import/{batchId}/failures.json  — Get Failures
GET /bulk/v1/customobjects/{apiName}/import/{batchId}/warnings.json  — Get Warnings
```

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/customobjects/car_c/import/4567/status.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Bulk Import Program Members

### Import Program Members

```
POST /bulk/v1/program/{programId}/members/import.json
```

Form parameters:
- `file` (required) — CSV/TSV/SSV file
- `format` (required) — `csv`, `tsv`, or `ssv`
- `programMemberStatus` (required) — the member status to assign (e.g. `Registered`, `Attended`)

```bash
curl -X POST "https://123-ABC-456.mktorest.com/bulk/v1/program/1010/members/import.json" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@program_members.csv" \
  -F "format=csv" \
  -F "programMemberStatus=Registered"
```

### Status, Failures, Warnings

```
GET /bulk/v1/program/members/import/{batchId}/status.json    — Get Status
GET /bulk/v1/program/members/import/{batchId}/failures.json  — Get Failures
GET /bulk/v1/program/members/import/{batchId}/warnings.json  — Get Warnings
```

```bash
curl "https://123-ABC-456.mktorest.com/bulk/v1/program/members/import/5678/status.json" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Full Export Workflow Example

End-to-end example exporting leads created in the last 30 days:

```bash
# 1. Create the export job
EXPORT_ID=$(curl -s -X POST "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": ["id", "email", "firstName", "lastName", "createdAt"],
    "format": "CSV",
    "filter": {
      "createdAt": {
        "startAt": "2025-06-01T00:00:00Z",
        "endAt": "2025-06-30T23:59:59Z"
      }
    }
  }' | jq -r '.result[0].exportId')

# 2. Enqueue the job
curl -s -X POST "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/${EXPORT_ID}/enqueue.json" \
  -H "Authorization: Bearer $TOKEN"

# 3. Poll until status is Completed
while true; do
  STATUS=$(curl -s "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/${EXPORT_ID}/status.json" \
    -H "Authorization: Bearer $TOKEN" | jq -r '.result[0].status')
  echo "Status: $STATUS"
  [ "$STATUS" = "Completed" ] && break
  [ "$STATUS" = "Failed" ] || [ "$STATUS" = "Cancelled" ] && echo "Job $STATUS" && exit 1
  sleep 60
done

# 4. Download the file
curl "https://123-ABC-456.mktorest.com/bulk/v1/leads/export/${EXPORT_ID}/file.json" \
  -H "Authorization: Bearer $TOKEN" \
  -o leads_export.csv
```

---

## Common Error Codes (Bulk API)

| Code | Meaning |
|------|---------|
| 601  | Access token invalid |
| 602  | Access token expired |
| 606  | Max rate limit exceeded |
| 607  | Daily quota reached |
| 1029 | Too many jobs in queue (10 concurrent limit) |
| 1035 | Unsupported filter type for the export |
| 1036 | Export file no longer available (expired after 30 days) |
