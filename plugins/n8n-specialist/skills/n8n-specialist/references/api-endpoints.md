# n8n REST API - Complete Endpoint Reference

Base path: `/api/v1`
Auth header: `X-N8N-API-KEY: <your-api-key>`
Content-Type: `application/json`

## Table of Contents

1. [Workflows](#workflows)
2. [Executions](#executions)
3. [Credentials](#credentials)
4. [Tags](#tags)
5. [Variables](#variables)
6. [Users](#users)
7. [Projects](#projects)
8. [Audit](#audit)
9. [Source Control](#source-control)
10. [Pagination](#pagination)
11. [Error Codes](#error-codes)

---

## Workflows

### GET /workflows

List all workflows with optional filters.

**Query parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `active` | boolean | Filter by active status |
| `tags` | string | Comma-separated tag names |
| `name` | string | Filter by workflow name (partial match) |
| `projectId` | string | Filter by project |
| `limit` | integer | Results per page (default 10, max 250) |
| `cursor` | string | Pagination cursor |
| `excludePinnedData` | boolean | Exclude pinned data from response |

**Response:**
```json
{
  "data": [
    {
      "id": "1234",
      "name": "My Workflow",
      "active": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-02T00:00:00.000Z",
      "tags": [{"id": "1", "name": "production"}],
      "nodes": [...],
      "connections": {...}
    }
  ],
  "nextCursor": "eyJpZCI6..."
}
```

### GET /workflows/{id}

Get a specific workflow by ID. Returns the full workflow definition including nodes, connections, and settings.

### POST /workflows

Create a new workflow.

**Request body:**
```json
{
  "name": "Workflow Name",
  "nodes": [
    {
      "id": "uuid",
      "name": "Start",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [250, 300],
      "parameters": {}
    }
  ],
  "connections": {},
  "settings": {
    "executionOrder": "v1"
  }
}
```

**Response:** The created workflow object with assigned `id`.

### PUT /workflows/{id}

Update an existing workflow. Send the full workflow definition.

**Important:** If the workflow is active, updating it will auto-republish with the new definition. Warn users editing production workflows.

### DELETE /workflows/{id}

Delete a workflow permanently. Cannot be undone.

### POST /workflows/{id}/activate

Activate (publish) a workflow. The workflow will start responding to its triggers.

**Response:** The updated workflow object with `active: true`.

### POST /workflows/{id}/deactivate

Deactivate a workflow. Stops it from responding to triggers.

**Response:** The updated workflow object with `active: false`.

### PUT /workflows/{id}/transfer

Transfer a workflow to a different project.

**Request body:**
```json
{
  "destinationProjectId": "project-id"
}
```

### GET /workflows/{id}/tags

Get tags assigned to a workflow.

### PUT /workflows/{id}/tags

Replace all tags on a workflow.

**Request body:**
```json
[
  {"id": "tag-id-1"},
  {"id": "tag-id-2"}
]
```

---

## Executions

### GET /executions

List executions with optional filters.

**Query parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `status` | string | `canceled`, `error`, `running`, `success`, `waiting` |
| `workflowId` | string | Filter by workflow |
| `projectId` | string | Filter by project |
| `includeData` | boolean | Include full execution data (default false) |
| `limit` | integer | Results per page |
| `cursor` | string | Pagination cursor |

**Response:**
```json
{
  "data": [
    {
      "id": "5678",
      "workflowId": "1234",
      "status": "success",
      "startedAt": "2024-01-01T12:00:00.000Z",
      "stoppedAt": "2024-01-01T12:00:05.000Z",
      "mode": "trigger"
    }
  ],
  "nextCursor": null
}
```

### GET /executions/{id}

Get a single execution. Add `?includeData=true` to get full node input/output data - essential for debugging.

### DELETE /executions/{id}

Delete an execution record.

### POST /executions/{id}/retry

Retry a failed or stopped execution.

**Request body:**
```json
{
  "loadWorkflow": true
}
```

- `loadWorkflow: true` - retry with the latest workflow version
- `loadWorkflow: false` - retry with the workflow snapshot from the original execution

---

## Credentials

### GET /credentials

List all credentials. **Secrets are never returned** - only metadata (id, name, type, createdAt, updatedAt).

### POST /credentials

Create a new credential.

**Request body:**
```json
{
  "name": "My API Key",
  "type": "httpHeaderAuth",
  "data": {
    "name": "Authorization",
    "value": "Bearer token123"
  }
}
```

Use `GET /credentials/schema/{typeName}` to discover the required fields for each credential type.

### PATCH /credentials/{id}

Update a credential. Only the credential owner can update.

### DELETE /credentials/{id}

Delete a credential.

### GET /credentials/schema/{credentialTypeName}

Get the JSON schema for a credential type, showing required and optional fields.

**Example:** `GET /credentials/schema/httpHeaderAuth`

### PUT /credentials/{id}/transfer

Transfer a credential to another project.

---

## Tags

### POST /tags

Create a tag.

**Request body:**
```json
{
  "name": "production"
}
```

### GET /tags

List all tags.

### GET /tags/{id}

Get a specific tag.

### PUT /tags/{id}

Update a tag name.

### DELETE /tags/{id}

Delete a tag.

---

## Variables

### GET /variables

List all variables.

### POST /variables

Create a variable.

**Request body:**
```json
{
  "key": "API_ENDPOINT",
  "value": "https://api.example.com"
}
```

### DELETE /variables/{id}

Delete a variable.

---

## Users

### GET /users

List all users. Only available to instance owners.

### POST /users

Create one or more users.

**Request body:**
```json
[
  {
    "email": "user@example.com",
    "role": "global:member"
  }
]
```

Roles: `global:owner`, `global:admin`, `global:member`

### GET /users/{id}

Get user by ID or email address.

### DELETE /users/{id}

Delete a user.

### PATCH /users/{id}/role

Change a user's global role.

**Request body:**
```json
{
  "newRoleName": "global:admin"
}
```

---

## Projects

n8n uses projects as the organizational unit (no folders - projects serve that role).

### GET /projects

List all projects.

**Query parameters:** `limit`, `cursor`

**Response:**
```json
{
  "data": [
    {
      "id": "project-id",
      "name": "My Project",
      "type": "team",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "nextCursor": null
}
```

### POST /projects

Create a new project.

**Request body:**
```json
{
  "name": "New Project"
}
```

### PUT /projects/{projectId}

Update a project.

**Request body:**
```json
{
  "name": "Renamed Project"
}
```

**Response:** 204 No Content

### DELETE /projects/{projectId}

Delete a project. **Response:** 204 No Content

### GET /projects/{projectId}/users

List members of a project and their roles.

**Query parameters:** `limit`, `cursor`

**Response:**
```json
{
  "data": [
    {
      "id": "user-id",
      "email": "user@example.com",
      "role": "project:editor"
    }
  ],
  "nextCursor": null
}
```

### POST /projects/{projectId}/users

Add users to a project.

**Request body:**
```json
[
  {
    "userId": "user-id",
    "role": "project:editor"
  }
]
```

### DELETE /projects/{projectId}/users/{userId}

Remove a user from a project. **Response:** 204 No Content

### PATCH /projects/{projectId}/users/{userId}

Change a user's role in a project.

**Request body:**
```json
{
  "role": "project:admin"
}
```

**Response:** 204 No Content

---

## Audit

### POST /audit

Generate a security audit report.

**Request body:**
```json
{
  "categories": ["credentials", "database", "nodes", "filesystem", "instance"]
}
```

---

## Source Control

### POST /source-control/pull

Pull workflows from a connected Git repository.

---

## Pagination

All list endpoints use cursor-based pagination.

**Response shape:**
```json
{
  "data": [...],
  "nextCursor": "eyJpZCI6..." | null
}
```

- Set `limit` to control page size
- Pass the `nextCursor` value as `cursor` in the next request
- When `nextCursor` is `null`, there are no more results

**Fetching all pages:**
```bash
CURSOR=""
while true; do
  RESPONSE=$(curl -s "${N8N_BASE_URL}/api/v1/workflows?limit=50${CURSOR:+&cursor=$CURSOR}" \
   -H "X-N8N-API-KEY: ${N8N_API_KEY}")
  echo "$RESPONSE" | jq '.data[]'
  CURSOR=$(echo "$RESPONSE" | jq -r '.nextCursor // empty')
  [ -z "$CURSOR" ] && break
done
```

---

## Error Codes

| Status | Meaning |
|--------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad request - invalid parameters |
| 401 | Unauthorized - invalid or missing API key |
| 403 | Forbidden - insufficient permissions |
| 404 | Not found - resource doesn't exist |
| 409 | Conflict - state incompatibility (e.g., activating an already active workflow) |
| 415 | Unsupported media type - wrong Content-Type header |
| 429 | Rate limited - slow down requests |
| 500 | Server error - n8n internal issue |
