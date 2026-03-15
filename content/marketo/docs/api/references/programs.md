# Programs & Smart Campaigns

Base URL: `https://{munchkin-id}.mktorest.com`
All requests require: `-H "Authorization: Bearer $TOKEN"`

Asset API POST endpoints use `application/x-www-form-urlencoded` unless noted.
Folder parameters accept JSON: `{"id":123,"type":"Folder"}`.

---

## Programs

### Browse Programs

```bash
curl -s "$BASE/rest/asset/v1/programs.json?maxReturn=20&offset=0" \
  -H "Authorization: Bearer $TOKEN"
```

Query params: `maxReturn` (max 200, default 20), `offset`, `filterType` (one of: id, name, type, channel, status, workspace), `filterValues` (comma-separated), `earliestUpdatedAt`, `latestUpdatedAt` (ISO 8601).

### Get Program by Id

```bash
curl -s "$BASE/rest/asset/v1/program/1001.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Program by Name

```bash
curl -s "$BASE/rest/asset/v1/program/byName.json?name=My%20Program" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Programs by Tag

```bash
curl -s "$BASE/rest/asset/v1/program/byTag.json?tagType=Channel&tagValue=Email" \
  -H "Authorization: Bearer $TOKEN"
```

### Create Program

```bash
curl -s "$BASE/rest/asset/v1/programs.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=New Program' \
  --data-urlencode 'folder={"id":18,"type":"Folder"}' \
  --data-urlencode 'description=Created via API' \
  --data-urlencode 'type=Default' \
  --data-urlencode 'channel=Email Send' \
  --data-urlencode 'costs=[{"startDate":"2026-01-01","cost":500}]' \
  --data-urlencode 'tags=[{"tagType":"Region","tagValue":"US"}]'
```

`type` values: `Default`, `Nurture`, `Event`, `Email`.

### Update Program Metadata

```bash
curl -s "$BASE/rest/asset/v1/program/1001.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'description=Updated description' \
  --data-urlencode 'name=Renamed Program' \
  --data-urlencode 'costs=[{"startDate":"2026-02-01","cost":1000}]' \
  --data-urlencode 'costsDestructiveUpdate=false'
```

### Delete Program

```bash
curl -s "$BASE/rest/asset/v1/program/1001/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

### Clone Program

```bash
curl -s "$BASE/rest/asset/v1/program/1001/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=Cloned Program' \
  --data-urlencode 'folder={"id":18,"type":"Folder"}' \
  --data-urlencode 'description=Clone of 1001'
```

### Approve Program

For Email programs only.

```bash
curl -s "$BASE/rest/asset/v1/program/1001/approve.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

---

## Smart Campaigns (Asset API)

### Browse Smart Campaigns

```bash
curl -s "$BASE/rest/asset/v1/smartCampaigns.json?maxReturn=20&offset=0" \
  -H "Authorization: Bearer $TOKEN"
```

Query params: `folder` (JSON), `maxReturn`, `offset`, `isActive` (boolean), `earliestUpdatedAt`, `latestUpdatedAt`.

### Get Smart Campaign by Id

```bash
curl -s "$BASE/rest/asset/v1/smartCampaign/1050.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Smart List by Smart Campaign Id

Returns the smart list (filter rules) associated with a smart campaign.

```bash
curl -s "$BASE/rest/asset/v1/smartCampaign/1050/smartList.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Activate Smart Campaign

Activates a trigger campaign. The campaign must have a trigger and an approved smart list.

```bash
curl -s "$BASE/rest/asset/v1/smartCampaign/1050/activate.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

### Deactivate Smart Campaign

```bash
curl -s "$BASE/rest/asset/v1/smartCampaign/1050/deactivate.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

### Clone Smart Campaign

```bash
curl -s "$BASE/rest/asset/v1/smartCampaign/1050/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=Cloned Campaign' \
  --data-urlencode 'folder={"id":18,"type":"Folder"}' \
  --data-urlencode 'description=Clone of 1050'
```

### Delete Smart Campaign

```bash
curl -s "$BASE/rest/asset/v1/smartCampaign/1050/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

---

## Campaigns (Lead Database API)

These endpoints are under `/rest/v1/` and use JSON request bodies.

### Get Campaigns

```bash
curl -s "$BASE/rest/v1/campaigns.json?batchSize=100" \
  -H "Authorization: Bearer $TOKEN"
```

Query params: `id` (comma-separated), `name` (comma-separated), `programName`, `workspaceName`, `batchSize` (max 300), `nextPageToken`, `isTriggerable` (boolean).

### Get Campaign by Id

```bash
curl -s "$BASE/rest/v1/campaigns/1050.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Request (Trigger) Campaign

Passes leads into a trigger campaign for immediate processing. Campaign must be active with a "Campaign is Requested" trigger.

```bash
curl -s "$BASE/rest/v1/campaigns/1050/trigger.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "input": {
      "leads": [{"id": 5001}, {"id": 5002}],
      "tokens": [
        {"name": "{{my.message}}", "value": "Welcome!"}
      ]
    }
  }'
```

### Schedule Campaign

Schedules a batch campaign to run at a future time. Optionally clones to a new program.

```bash
curl -s "$BASE/rest/v1/campaigns/1050/schedule.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "input": {
      "runAt": "2026-04-01T10:00:00Z",
      "tokens": [
        {"name": "{{my.subject}}", "value": "April Offer"}
      ],
      "cloneToProgramName": "April Campaign Clone"
    }
  }'
```

`runAt` is ISO 8601. If omitted, campaign runs within 5 minutes. `cloneToProgramName` creates a copy of the parent program before running.

---

## Smart Lists

### Browse Smart Lists

```bash
curl -s "$BASE/rest/asset/v1/smartLists.json?maxReturn=20&offset=0" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Smart List by Id

```bash
curl -s "$BASE/rest/asset/v1/smartList/3001.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Smart List by Name

```bash
curl -s "$BASE/rest/asset/v1/smartList/byName.json?name=My%20Smart%20List" \
  -H "Authorization: Bearer $TOKEN"
```

### Clone Smart List

```bash
curl -s "$BASE/rest/asset/v1/smartList/3001/clone.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=Cloned Smart List' \
  --data-urlencode 'folder={"id":18,"type":"Folder"}' \
  --data-urlencode 'description=Clone of 3001'
```

### Delete Smart List

```bash
curl -s "$BASE/rest/asset/v1/smartList/3001/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

---

## Tokens

Tokens are program-level variables (My Tokens) that can be set on folders or programs.

### Get Tokens by Folder Id

```bash
curl -s "$BASE/rest/asset/v1/folder/1001/tokens.json?folderType=Program" \
  -H "Authorization: Bearer $TOKEN"
```

`folderType`: `Folder` or `Program`.

### Create Token

```bash
curl -s "$BASE/rest/asset/v1/folder/1001/tokens.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'folderType=Program' \
  --data-urlencode 'name=Subject Line' \
  --data-urlencode 'type=text' \
  --data-urlencode 'value=March Newsletter'
```

Common token types: `text`, `date`, `rich text`, `score`, `campaign`, `number`.

### Delete Token

```bash
curl -s "$BASE/rest/asset/v1/folder/1001/tokens/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=Subject Line' \
  --data-urlencode 'type=text' \
  --data-urlencode 'folderType=Program'
```

---

## Tags & Channels

### Get Tag Types

```bash
curl -s "$BASE/rest/asset/v1/tagTypes.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Tag Type by Name

```bash
curl -s "$BASE/rest/asset/v1/tagType/byName.json?name=Region" \
  -H "Authorization: Bearer $TOKEN"
```

Returns allowed values for the tag type.

### Get Channels

```bash
curl -s "$BASE/rest/asset/v1/channels.json?maxReturn=200" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Channel by Name

```bash
curl -s "$BASE/rest/asset/v1/channel/byName.json?name=Email%20Send" \
  -H "Authorization: Bearer $TOKEN"
```

Returns progression statuses for the channel.

---

## Segments & Snippets

### Get Segmentations

```bash
curl -s "$BASE/rest/asset/v1/segmentation.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Segments

```bash
curl -s "$BASE/rest/asset/v1/segmentation/2001/segments.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Browse Snippets

```bash
curl -s "$BASE/rest/asset/v1/snippets.json?maxReturn=20&offset=0" \
  -H "Authorization: Bearer $TOKEN"
```

### Get Snippet by Id

```bash
curl -s "$BASE/rest/asset/v1/snippet/4001.json" \
  -H "Authorization: Bearer $TOKEN"
```

### Create Snippet

```bash
curl -s "$BASE/rest/asset/v1/snippets.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=Footer Snippet' \
  --data-urlencode 'folder={"id":18,"type":"Folder"}' \
  --data-urlencode 'description=Shared footer'
```

### Update Snippet

```bash
curl -s "$BASE/rest/asset/v1/snippet/4001.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=Updated Footer' \
  --data-urlencode 'description=Revised footer snippet'
```

### Delete Snippet

```bash
curl -s "$BASE/rest/asset/v1/snippet/4001/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

---

## Folders & Files

### Browse Folders

```bash
curl -s "$BASE/rest/asset/v1/folders.json?root=18&maxDepth=3&maxReturn=20" \
  -H "Authorization: Bearer $TOKEN"
```

Query params: `root` (folder id), `maxDepth`, `maxReturn`, `offset`, `workspace`.

### Get Folder by Id

```bash
curl -s "$BASE/rest/asset/v1/folder/18.json?type=Folder" \
  -H "Authorization: Bearer $TOKEN"
```

`type`: `Folder` or `Program`.

### Get Folder by Name

```bash
curl -s "$BASE/rest/asset/v1/folder/byName.json?name=Emails&type=Folder" \
  -H "Authorization: Bearer $TOKEN"
```

### Create Folder

```bash
curl -s "$BASE/rest/asset/v1/folders.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  --data-urlencode 'name=New Folder' \
  --data-urlencode 'parent={"id":18,"type":"Folder"}' \
  --data-urlencode 'description=Created via API'
```

### Delete Folder

Folder must be empty.

```bash
curl -s "$BASE/rest/asset/v1/folder/99/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST
```

### Browse Files

```bash
curl -s "$BASE/rest/asset/v1/files.json?maxReturn=20&offset=0" \
  -H "Authorization: Bearer $TOKEN"
```

### Create File (Upload)

Uses `multipart/form-data`.

```bash
curl -s "$BASE/rest/asset/v1/files.json" \
  -H "Authorization: Bearer $TOKEN" \
  -X POST \
  -F 'name=banner.png' \
  -F 'folder={"id":18,"type":"Folder"}' \
  -F 'file=@/path/to/banner.png' \
  -F 'description=Homepage banner' \
  -F 'insertOnly=false'
```

`insertOnly=true` prevents overwriting an existing file with the same name.

---

## Common Response Shape

All endpoints return:

```json
{
  "success": true,
  "errors": [],
  "warnings": [],
  "requestId": "a1b2#17c3d4e5f6",
  "result": [ ... ]
}
```

On error, `success` is `false` and `errors` contains `[{"code": "...", "message": "..."}]`.

## Common Error Codes

| Code | Meaning |
|------|---------|
| 601  | Access token empty / invalid |
| 602  | Access token expired |
| 603  | Access denied (insufficient permissions) |
| 608  | API temporarily unavailable |
| 611  | System error |
| 701  | Asset name cannot be blank |
| 702  | Asset not found |
| 709  | Folder not found |
| 710  | Parent folder not found |

## Pagination

Asset API endpoints use `offset` + `maxReturn` (max 200). Lead Database API endpoints (`/rest/v1/campaigns.json`) use `batchSize` (max 300) + `nextPageToken`. Check `moreResult: true` in the response to know if more pages exist.
