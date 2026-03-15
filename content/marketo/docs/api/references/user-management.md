# User Management

Manage users, roles, and workspaces in Marketo Engage via the REST API.

**Base URL:** `https://{munchkin-id}.mktorest.com`

All requests require the header `-H "Authorization: Bearer $TOKEN"`.

**Required permissions:** Access User Management Api AND Access Users.

**Important:** The `{userid}` parameter is the user's email address for Marketo Identity users. For Adobe Identity users, use the generated GUID. User Management endpoints accept JSON request bodies (`Content-Type: application/json`), not form-encoded.

---

## Users

### Get Users

Returns a paginated list of all users.

```bash
curl -s "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/allusers.json?pageOffset=0&pageSize=50" \
  -H "Authorization: Bearer $TOKEN"
```

**Query parameters:**

| Parameter | Type | Description |
|---|---|---|
| pageOffset | integer | Page offset for pagination (default 0) |
| pageSize | integer | Results per page, max 200, default 20 |

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "jdoe@example.com",
      "firstName": "Jane",
      "lastName": "Doe",
      "emailAddress": "jdoe@example.com",
      "id": "jdoe@example.com"
    }
  ]
}
```

### Get User by Id

```bash
curl -s "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/jdoe@example.com/user.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "jdoe@example.com",
      "firstName": "Jane",
      "lastName": "Doe",
      "emailAddress": "jdoe@example.com",
      "apiOnly": false
    }
  ]
}
```

### Invite User

Creates and sends an invitation to a new user.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/invite.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "emailAddress": "newuser@example.com",
    "firstName": "Alex",
    "lastName": "Smith",
    "userRoleWorkspaces": [
      {
        "accessRoleId": 1,
        "workspaceId": 1
      }
    ],
    "apiOnly": false,
    "expiresAt": "2026-12-31T23:59:59Z",
    "reason": "New marketing team member"
  }'
```

**Request body fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| emailAddress | string | yes | User's email |
| firstName | string | yes | First name |
| lastName | string | yes | Last name |
| userRoleWorkspaces | array | yes | Array of `{accessRoleId, workspaceId}` |
| apiOnly | boolean | no | If true, user is API-only (no UI access) |
| expiresAt | string | no | ISO 8601 expiration date for API-only users |
| reason | string | no | Reason for the invitation |

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "newuser@example.com",
      "firstName": "Alex",
      "lastName": "Smith",
      "emailAddress": "newuser@example.com"
    }
  ]
}
```

### Get Invited User by Id

Retrieves a pending (not yet accepted) invited user.

```bash
curl -s "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/newuser@example.com/invite.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "newuser@example.com",
      "firstName": "Alex",
      "lastName": "Smith",
      "emailAddress": "newuser@example.com",
      "status": "pending"
    }
  ]
}
```

### Update User Attributes

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/jdoe@example.com/update.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Janet",
    "lastName": "Doe",
    "emailAddress": "jdoe@example.com",
    "apiOnly": false
  }'
```

**Request body fields:**

| Field | Type | Description |
|---|---|---|
| firstName | string | Updated first name |
| lastName | string | Updated last name |
| emailAddress | string | Updated email |
| apiOnly | boolean | Toggle API-only access |
| expiresAt | string | ISO 8601 expiration (API-only users) |

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "jdoe@example.com",
      "firstName": "Janet",
      "lastName": "Doe"
    }
  ]
}
```

### Delete User

Permanently removes an active user.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/jdoe@example.com/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "jdoe@example.com"
    }
  ]
}
```

### Delete Invited User

Removes a pending invitation that has not yet been accepted.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/newuser@example.com/invite/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "userid": "newuser@example.com"
    }
  ]
}
```

---

## Roles

### Get Roles

Returns all available roles in the subscription.

```bash
curl -s "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/roles.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "id": 1,
      "name": "Admin",
      "description": "Full administrative access",
      "type": "system",
      "isHidden": false,
      "isOnlyAllZones": true,
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-06-20T14:30:00Z"
    },
    {
      "id": 2,
      "name": "Marketing User",
      "description": "Standard marketing user role",
      "type": "system",
      "isHidden": false,
      "isOnlyAllZones": false,
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-06-20T14:30:00Z"
    }
  ]
}
```

### Get Roles and Workspaces by User Id

```bash
curl -s "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/jdoe@example.com/roles.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "accessRoleId": 1,
      "roleName": "Admin",
      "workspaceId": 1,
      "workspaceName": "Default"
    }
  ]
}
```

### Add Roles

Assigns one or more role-workspace pairs to a user.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/jdoe@example.com/roles/create.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [
      {
        "accessRoleId": 2,
        "workspaceId": 1
      },
      {
        "accessRoleId": 1,
        "workspaceId": 3
      }
    ]
  }'
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "accessRoleId": 2,
      "workspaceId": 1
    },
    {
      "accessRoleId": 1,
      "workspaceId": 3
    }
  ]
}
```

### Delete Roles

Removes one or more role-workspace pairs from a user.

```bash
curl -s -X POST "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/jdoe@example.com/roles/delete.json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [
      {
        "accessRoleId": 2,
        "workspaceId": 1
      }
    ]
  }'
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "accessRoleId": 2,
      "workspaceId": 1
    }
  ]
}
```

---

## Workspaces

### Get Workspaces

Returns all workspaces in the subscription.

```bash
curl -s "https://{munchkin-id}.mktorest.com/userservice/management/v1/users/workspaces.json" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**

```json
{
  "success": true,
  "result": [
    {
      "id": 1,
      "name": "Default",
      "description": "Primary workspace",
      "status": "active",
      "currencyInfo": {
        "code": "USD",
        "symbol": "$"
      },
      "globalViz": 0
    },
    {
      "id": 3,
      "name": "EMEA",
      "description": "Europe workspace",
      "status": "active",
      "currencyInfo": {
        "code": "EUR",
        "symbol": "€"
      },
      "globalViz": 1
    }
  ]
}
```

---

## HTTP Response Codes

| Code | Meaning |
|---|---|
| 200 | OK — request succeeded |
| 401 | Unauthorized — invalid or expired token |
| 403 | Forbidden — insufficient permissions |
| 404 | Not Found — user or resource does not exist |
