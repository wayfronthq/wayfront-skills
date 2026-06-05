# Wayfront MCP Server

Connect an AI agent (or any MCP client) to a workspace to query and act on its data, within the permissions of the staff account it connects as. Endpoint:
`https://[WORKSPACE].wayfront.com/mcp`, Streamable HTTP, OAuth 2.1.

**Two prerequisites:** the **MCP module must be enabled** in the workspace (Settings, Modules), and you must connect as a **staff account**. Client accounts are refused (403); the MCP is a staff/agent surface, not a client one. Until the module is on, connections fail.

## Connection Setup

Wayfront supports OAuth 2.1 with dynamic client registration. Clients that speak MCP OAuth (Claude Code, Claude Desktop, Cursor) authorize automatically. You only need the workspace URL.

### Claude Code
```bash
claude mcp add --transport http wayfront https://[WORKSPACE].wayfront.com/mcp
```
`--transport http` is required. The default transport is stdio, which would try to run a local process and fail. The browser opens to authorize on first tool use.

### Claude Desktop
1. **Settings, MCP Servers, Add**
2. Server URL: `https://[WORKSPACE].wayfront.com/mcp`
3. Authorize in the browser when prompted.

### Any MCP client (Streamable HTTP + OAuth)
- **Endpoint:** `https://[WORKSPACE].wayfront.com/mcp`
- **Transport:** Streamable HTTP. Client messages use `POST`; `GET` exists only to return a spec-compliant `405`.
- **Auth:** OAuth 2.1, discovery via `/.well-known/oauth-authorization-server`, scope `mcp:use`

In JSON config form, declare the transport explicitly:
```json
{ "wayfront": { "type": "http", "url": "https://[WORKSPACE].wayfront.com/mcp" } }
```

### Headless clients
The tenant MCP authenticates with OAuth (scope `mcp:use`). If a runtime can't complete an interactive browser flow, use a proxy that completes and caches the OAuth session for it. Either way it runs as a staff account, whose permissions decide what each tool can do.

(There is a separate internal `/mcp/support` server for Wayfront's own staff that uses token auth and requires a `tenant_id` on every call. It is management-only and IP-restricted. Workspace agents never use it: connect to `/mcp` and do not pass `tenant_id`.)

## How the Tools Are Shaped

This is a real action surface for AI agents, not just a reporting feed. Agents use it to look things up and to act, within what the connected account is allowed to do. Today many tools are read-only (`list`/`show`) and the set with write actions keeps growing, so check the schema rather than assuming a ceiling. If a change you need is not exposed as a tool action, say that the MCP surface does not currently support it instead of inventing a workaround.

Tools are named by entity, for example `OrderTool`, `InvoiceTool`, `ServiceTool`, `ClientTool`. The exact set grows over time, so **read `tools/list` for the current tools**. The shape is consistent:

- Most tools expose `list` (query) and `show` (one record).
- A growing number also expose writes (`create`, `update`, and similar). Don't assume a tool is read-only; check its schema for write actions.
- Nested tools (an order's messages, an order's tasks, a client's activity) take a parent identifier.
- Each tool's schema declares its actions, its id parameter, its filterable and sortable fields, and its status code map. Treat `tools/list` as the capability surface, not an authorization contract: permission, ownership, module, and policy checks run when you call, so an action visible in the schema can still be refused at call time.

**Do not assume actions, fields, statuses, or permissions from memory.** They change, and several are customized per workspace. The schema is the source of truth for shape; the call result is the source of truth for what you're allowed to do.

## Permissions & Access Control

The MCP acts as the staff account that authorized the connection, and inherits exactly that account's role and permissions. An admin account can do everything; a limited role can only use the tools and actions its permissions allow (a tool returns a permission error otherwise). Only staff can connect at all: a client account is refused at the door.

So you control an AI agent by controlling the account it connects as:

- **Create a dedicated staff account for the agent** and give it a role scoped to just what it should do. Connect the agent as that account rather than as a human admin.
- **Widen or narrow the role** to change the agent's reach. Removing a permission removes the matching tool actions; granting one adds them.
- **Revoke the account** (or its session) to cut the agent off entirely.

This also keeps the audit trail clean: actions are attributed to the agent's account, not to a person.

## Reading: Filter, Sort, Paginate

`list` accepts `filters`, `sort`, `limit`, and `page`.

### Filters
Keys are field names from the tool's schema, values are operator objects. Common operators (the full set is larger and can change, so confirm against the live operator list):

| Operator | Meaning |
|---|---|
| `$eq` / `$ne` | equals / not equals |
| `$lt` `$gt` `$lte` `$gte` | comparisons |
| `$in` / `$notIn` | in / not in a list |
| `$contains` / `$notContains` | substring |
| `$startsWith` / `$endsWith` | prefix / suffix |
| `$null` / `$notNull` | null checks |
| `$between` | range |
| `$and` / `$or` | compound groups |

```json
{"filters": {"$and": [{"<field>": {"$eq": "<value>"}}, {"<date_field>": {"$lt": "<iso>"}}]}}
```

Case-sensitive variants and relation filters also exist. Confirm the live operator and field set in the schema. **Field names are model-specific and not guessable**, so list a record first or read the schema before filtering.

### Status filters take the numeric code
Status is an integer, not a label. Filter with the code (`{"<status_field>": {"$eq": <code>}}`), never the name. The code-to-label map is printed in the tool's `filters` description and is customizable per workspace. Read it there.

### Sort and pagination
`sort` is an array of fields with optional `:asc` or `:desc` (default `:asc`), e.g. `["created_at:desc"]`. Pagination is `limit` and `page`; the response carries `pagination.{current_page,last_page,per_page,total}`. Iterate while `current_page < last_page`.

## Working Pattern

The reliable loop for any task:

1. `tools/list` to pick the right entity tool and confirm its actions.
2. Read that tool's schema for the id parameter, fields, and status map.
3. `list` with a filter to find records (and to discover identifiers you don't have).
4. `show` for detail, or a write action if the tool supports one and you have permission.
5. If the MCP does not expose the needed change, report that limitation and stop unless the user explicitly asks to use another surface.

Identifiers: most tools take a numeric id, but orders and tickets take their string `number`. Nested tools take the parent's identifier. The schema names the exact parameter.

## Error Handling

| Error | Meaning | Action |
|---|---|---|
| not found | record missing or out of scope | verify the id/number, `list` with a strict filter first |
| unsupported field | filtered or sorted on a field not in the schema | use only fields the schema lists |
| permission error | the staff role lacks the tool's permission | enable that permission in Wayfront settings |
| action not available | a write was attempted where unsupported | check the schema; report that the MCP tool does not expose that action |
| empty `data: []` | no matching records | loosen filters or confirm data exists |

Results are scoped to what the connected staff account's role allows.

## Efficiency

- No published hard rate limit, but avoid tight loops of hundreds of sequential calls.
- Raise the page limit and paginate rather than fetching one record at a time.
- For bulk questions ("all clients with unpaid invoices"), use one filtered `list`.
- Fetch a parent once, then query its children by its identifier.

## Tips

- The MCP is a working surface for agents, not just reporting. It can act wherever the connected account has the permission and the tool exposes the action; if it does not expose an action, do not assume another surface should be used automatically.
- Scope an agent by connecting it as a dedicated staff account with a limited role (see Permissions & Access Control).
- Status is always a number. Read the live code map from the tool description.
- `is_modified: true` on a template means it was customised away from its default (see `templates.md`).
- The log and activity tools are the audit trail: what happened and when.
