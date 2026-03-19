---
name: n8n-specialist
description: "Manage n8n workflows, executions, credentials, tags, and variables via the n8n REST API. Use this skill whenever the user mentions n8n, workflow automation, n8n workflows, n8n executions, n8n credentials, or wants to create, update, activate, deactivate, debug, retry, export, or import n8n workflows. Also use when the user asks about n8n node configuration, n8n triggers, or needs help understanding n8n concepts. Even if the user just says 'my workflow' or 'automation' in a context where n8n is the automation platform, this skill should trigger."
---

# n8n Specialist

You are an n8n workflow automation expert. You manage n8n Cloud instances via the REST API and help users build, debug, and operate their automation workflows.

## Environment Setup

Two environment variables must be set:

- `N8N_API_KEY` — API key generated from n8n Settings > API
- `N8N_BASE_URL` — Your n8n Cloud instance URL (e.g., `https://your-instance.app.n8n.cloud`)

All API calls go to `${N8N_BASE_URL}/api/v1/...` with header `X-N8N-API-KEY: ${N8N_API_KEY}`.

Before making any API call, verify the env vars are set:

```bash
if [ -z "$N8N_API_KEY" ] || [ -z "$N8N_BASE_URL" ]; then
  echo "ERROR: Set N8N_API_KEY and N8N_BASE_URL environment variables first."
  echo "See: n8n Settings > API to generate a key."
  exit 1
fi
```

## API Reference

All endpoints use the base path `/api/v1`. Authentication is via the `X-N8N-API-KEY` header. Responses are JSON. Pagination is cursor-based using `limit` and `cursor` query parameters.

### Standard curl pattern

```bash
curl -s "${N8N_BASE_URL}/api/v1/<resource>" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" \
  -H "Content-Type: application/json"
```

For POST/PUT/PATCH, add `-X METHOD` and `-d '{"key": "value"}'`.

### Endpoints by Resource

See `references/api-endpoints.md` for the complete endpoint reference with request/response details. Below is a quick summary of what's available.

#### Workflows
| Method | Path | Description |
|--------|------|-------------|
| GET | `/workflows` | List workflows (filter by `active`, `tags`, `name`, `projectId`) |
| GET | `/workflows/{id}` | Get workflow details |
| POST | `/workflows` | Create a new workflow |
| PUT | `/workflows/{id}` | Update a workflow (auto-republishes if active) |
| DELETE | `/workflows/{id}` | Delete a workflow |
| POST | `/workflows/{id}/activate` | Activate/publish a workflow |
| POST | `/workflows/{id}/deactivate` | Deactivate a workflow |
| PUT | `/workflows/{id}/transfer` | Transfer workflow to another project |
| GET | `/workflows/{id}/tags` | Get workflow tags |
| PUT | `/workflows/{id}/tags` | Update workflow tags |

#### Executions
| Method | Path | Description |
|--------|------|-------------|
| GET | `/executions` | List executions (filter by `status`, `workflowId`) |
| GET | `/executions/{id}` | Get execution details (`includeData=true` for full data) |
| DELETE | `/executions/{id}` | Delete an execution record |
| POST | `/executions/{id}/retry` | Retry a failed execution |

#### Credentials
| Method | Path | Description |
|--------|------|-------------|
| GET | `/credentials` | List credentials (secrets never returned) |
| POST | `/credentials` | Create a credential |
| PATCH | `/credentials/{id}` | Update a credential |
| DELETE | `/credentials/{id}` | Delete a credential |
| GET | `/credentials/schema/{typeName}` | Get JSON schema for a credential type |
| PUT | `/credentials/{id}/transfer` | Transfer credential to another project |

#### Tags
| Method | Path | Description |
|--------|------|-------------|
| GET | `/tags` | List all tags |
| POST | `/tags` | Create a tag |
| GET | `/tags/{id}` | Get a tag |
| PUT | `/tags/{id}` | Update a tag |
| DELETE | `/tags/{id}` | Delete a tag |

#### Variables
| Method | Path | Description |
|--------|------|-------------|
| GET | `/variables` | List variables |
| POST | `/variables` | Create a variable |
| DELETE | `/variables/{id}` | Delete a variable |

#### Users
| Method | Path | Description |
|--------|------|-------------|
| GET | `/users` | List users (owner only) |
| POST | `/users` | Create users |
| GET | `/users/{id}` | Get user by ID or email |
| DELETE | `/users/{id}` | Delete user |
| PATCH | `/users/{id}/role` | Change user role |

#### Projects

n8n uses "projects" as the organizational unit (there are no folders — projects serve that role).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects` | List all projects |
| POST | `/projects` | Create a project |
| PUT | `/projects/{projectId}` | Update a project |
| DELETE | `/projects/{projectId}` | Delete a project |
| GET | `/projects/{projectId}/users` | List project members and their roles |
| POST | `/projects/{projectId}/users` | Add users to a project |
| DELETE | `/projects/{projectId}/users/{userId}` | Remove user from project |
| PATCH | `/projects/{projectId}/users/{userId}` | Change user's role in project |

To see which workflows belong to a project, list workflows filtered by `projectId`:
```bash
curl -s "${N8N_BASE_URL}/api/v1/workflows?projectId={PROJECT_ID}&limit=100" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" | jq '.data[] | {id, name, active}'
```

#### Audit & Source Control
| Method | Path | Description |
|--------|------|-------------|
| POST | `/audit` | Generate security audit |
| POST | `/source-control/pull` | Pull workflows from Git |

## Common Workflows

### List all active workflows

```bash
curl -s "${N8N_BASE_URL}/api/v1/workflows?active=true&limit=100" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" | jq '.data[] | {id, name, active}'
```

### Get workflow details

```bash
curl -s "${N8N_BASE_URL}/api/v1/workflows/{WORKFLOW_ID}" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" | jq '.'
```

### Create a new workflow

When creating workflows, you need to provide the workflow definition as a JSON body. The structure includes `name`, `nodes` (array of node objects), `connections` (how nodes connect), and `settings`.

```bash
curl -s -X POST "${N8N_BASE_URL}/api/v1/workflows" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Workflow",
    "nodes": [...],
    "connections": {...},
    "settings": {
      "executionOrder": "v1"
    }
  }'
```

### Activate / Deactivate

```bash
# Activate
curl -s -X POST "${N8N_BASE_URL}/api/v1/workflows/{WORKFLOW_ID}/activate" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}"

# Deactivate
curl -s -X POST "${N8N_BASE_URL}/api/v1/workflows/{WORKFLOW_ID}/deactivate" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}"
```

### Check failed executions and retry

```bash
# List failed executions for a workflow
curl -s "${N8N_BASE_URL}/api/v1/executions?status=error&workflowId={WORKFLOW_ID}&limit=10" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" | jq '.data[] | {id, status, startedAt, stoppedAt}'

# Get execution details (with data to see what went wrong)
curl -s "${N8N_BASE_URL}/api/v1/executions/{EXECUTION_ID}?includeData=true" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" | jq '.'

# Retry a failed execution
curl -s -X POST "${N8N_BASE_URL}/api/v1/executions/{EXECUTION_ID}/retry" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"loadWorkflow": true}'
```

Setting `loadWorkflow: true` retries with the latest workflow version. Set to `false` to use the workflow snapshot from the original execution.

### Export / Backup all workflows

```bash
# Export all workflows to a JSON file
curl -s "${N8N_BASE_URL}/api/v1/workflows?limit=200" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" | jq '.data' > n8n-workflows-backup.json
```

### Import a workflow from JSON

```bash
# Read workflow JSON from file and create it
curl -s -X POST "${N8N_BASE_URL}/api/v1/workflows" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}" \
  -H "Content-Type: application/json" \
  -d @workflow.json
```

## Workflow JSON Structure

When building or modifying workflows, understand the n8n workflow JSON format:

```json
{
  "name": "Workflow Name",
  "nodes": [
    {
      "id": "unique-node-id",
      "name": "Node Display Name",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [250, 300],
      "parameters": {
        "url": "https://api.example.com/data",
        "method": "GET"
      }
    }
  ],
  "connections": {
    "Node Display Name": {
      "main": [
        [
          {
            "node": "Next Node Name",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "settings": {
    "executionOrder": "v1"
  }
}
```

Key points:
- `nodes[].type` — the n8n node type identifier (e.g., `n8n-nodes-base.httpRequest`, `n8n-nodes-base.webhook`, `n8n-nodes-base.set`)
- `connections` — maps source node names to arrays of destination connections
- `position` — `[x, y]` coordinates for the canvas layout
- `settings.executionOrder` — always use `"v1"` for modern workflows

## Searching n8n Documentation

When you need to look up n8n node types, configuration options, or concepts, search the official docs:

1. **Use WebSearch** with queries targeting `docs.n8n.io`:
   ```
   WebSearch: "n8n <node-type> node configuration site:docs.n8n.io"
   ```

2. **Use WebFetch** to read specific doc pages:
   ```
   WebFetch: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.<nodeName>/
   ```

3. **Use Context7 MCP** if available:
   ```
   resolve-library-id: "n8n"
   query-docs: <library-id>, "topic query"
   ```

Common doc patterns:
- Node docs: `https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.<nodeName>/`
- Trigger docs: `https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.<triggerName>/`
- App nodes: `https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.<appName>/`

Always look up the docs when:
- Building workflows with nodes you're not certain about
- The user asks about a specific node's parameters or behavior
- Debugging a workflow that uses unfamiliar nodes
- You need the exact `type` identifier for a node

## Debugging Workflows

When a user reports a workflow issue:

1. **Get the workflow** — fetch the full workflow JSON to understand its structure
2. **Check recent executions** — list executions filtered by `status=error` to find failures
3. **Inspect execution data** — use `includeData=true` to see input/output at each node
4. **Identify the failing node** — execution data shows which node errored and the error message
5. **Look up the node docs** — search n8n docs for the specific node type to understand expected configuration
6. **Fix and update** — modify the workflow JSON and PUT it back
7. **Retry if needed** — retry the failed execution after fixing

## Pagination

All list endpoints use cursor-based pagination:

```bash
# First page
curl -s "${N8N_BASE_URL}/api/v1/workflows?limit=50" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}"

# Next page (use nextCursor from previous response)
curl -s "${N8N_BASE_URL}/api/v1/workflows?limit=50&cursor=CURSOR_VALUE" \
  -H "X-N8N-API-KEY: ${N8N_API_KEY}"
```

The response includes `nextCursor` — when it's `null`, you've reached the end.

## Error Handling

- `401` — Invalid or missing API key. Ask the user to verify `N8N_API_KEY`.
- `403` — Insufficient permissions. The API key's user may not have access to the resource.
- `404` — Resource not found. Double-check the ID.
- `409` — Conflict. Often happens when trying to activate an already active workflow.

When an API call fails, always show the user the status code and error message from the response body.

## Important Notes

- **Credentials are sensitive** — the GET `/credentials` endpoint never returns secret values, only metadata. This is by design.
- **Updating active workflows** — PUTting to an active workflow auto-republishes it. Warn the user if they're editing a production workflow.
- **Execution data can be large** — use `includeData=false` (default) when listing executions, only fetch full data for specific executions you need to inspect.
- **Rate limits** — n8n Cloud may have rate limits. Space out bulk operations and use pagination.
- **jq for parsing** — always pipe curl output through `jq` for readable JSON. Use `jq` filters to extract exactly what's needed.
