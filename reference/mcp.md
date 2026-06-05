# Wayfront MCP Server

Query and act on workspace data from any MCP client. Endpoint:
`https://[WORKSPACE].wayfront.com/mcp` — Streamable HTTP (POST), OAuth 2.1.

## Connection Setup

Wayfront supports OAuth 2.1 with dynamic client registration. Clients that speak MCP OAuth (Claude Code, Claude Desktop, Cursor) authorize automatically — you only need the workspace URL.

### Claude Code
```bash
claude mcp add --transport http wayfront https://[WORKSPACE].wayfront.com/mcp
```
The browser opens to authorize on first tool use.

### Claude Desktop
1. **Settings → MCP Servers → Add**
2. Server URL: `https://[WORKSPACE].wayfront.com/mcp`
3. Authorize in the browser when prompted.

### Any MCP client (Streamable HTTP + OAuth)
- **Endpoint:** `https://[WORKSPACE].wayfront.com/mcp`
- **Transport:** Streamable HTTP (POST only — a `405` on `GET` is spec-compliant)
- **Auth:** OAuth 2.1, discovery via `/.well-known/oauth-authorization-server`, scope `mcp:use`

### Headless / token-based clients (e.g. OpenClaw via mcporter)
Some runtimes can't complete an interactive OAuth browser flow. Use a middleware that caches the OAuth token (mcporter), or a static MCP token where the workspace supports one (`WAYFRONT_MCP_TOKEN`). The token's staff permissions determine tool access.

## Discover Tools at Runtime

**This server is schema-first.** Before calling a tool, read its description and input schema from `tools/list`. Each tool tells you:
- which entity it queries,
- which actions it supports (`list`, `show`, and sometimes `update`/`create`),
- which fields are filterable and sortable.

New endpoints appear automatically — never assume a fixed tool list. The live schema is authoritative.

## Tool Inventory

Entity tools — most support `list` (query with filters/sort/pagination) and `show` (one record by id/number). Write actions (`create`/`update`) are exposed where your permissions allow; a read-only support deployment blocks writes.

| Tool | Identified by | Notes |
|---|---|---|
| `ClientTool` | `id` | clients with addresses, companies, managers, signup params; `update` can set the client note |
| `OrderTool` | `number` (`ORD-42`) | statuses, tags, assignees, filled form fields |
| `OrderMessageTool` | parent order number | messages on an order |
| `OrderTaskTool` | parent order number | tasks/checklist on an order |
| `TicketTool` | `number` (`TKT-7`) | statuses, tags, assignees |
| `TicketMessageTool` | parent ticket number | messages on a ticket |
| `InvoiceTool` | `id` | filter by status, payment system, dates |
| `SubscriptionTool` | `id` | active recurring plans |
| `ServiceTool` | `id` | the product catalogue (see `services.md`) |
| `FormTool` | `id` | forms and their fields |
| `CouponTool` | `id` | discount codes |
| `TemplateTool` | `name` | DB overrides + filesystem defaults (see `templates.md`) |
| `PipelineTool` / `PipelineItemTool` | `id` | sales/production pipelines and cards |
| `TeamTool` | user `id` | staff accounts |
| `TagTool` | — | returns all tags at once |
| `LogTool` / `UserActivityTool` | — | audit/debug trails (list only) |

There are also tenant-info and Stripe diagnostic tools on support deployments. Always check `tools/list` for the exact set available to your token.

## Purity Filter Syntax

Most `list` actions accept a `filters` object using Purity-style operators.

| Operator | Meaning | Example |
|---|---|---|
| `$eq` | Equals | `{"status": {"$eq": "active"}}` |
| `$ne` | Not equals | `{"status": {"$ne": "cancelled"}}` |
| `$lt` / `$gt` | Less / greater than | `{"amount": {"$gt": 100}}` |
| `$lte` / `$gte` | Less / greater than or equal | `{"amount": {"$gte": 50}}` |
| `$in` | In array | `{"status": {"$in": ["active", "pending"]}}` |
| `$notIn` | Not in array | `{"status": {"$notIn": ["archived"]}}` |
| `$contains` | String contains | `{"name": {"$contains": "john"}}` |
| `$notContains` | String does not contain | `{"email": {"$notContains": "test"}}` |
| `$startsWith` | Starts with | `{"email": {"$startsWith": "admin"}}` |
| `$endsWith` | Ends with | `{"email": {"$endsWith": ".com"}}` |
| `$null` | Is null | `{"deleted_at": {"$null": true}}` |
| `$notNull` | Is not null | `{"paid_at": {"$notNull": true}}` |
| `$between` | Between two values | `{"amount": {"$between": [100, 500]}}` |
| `$or` | OR compound | `{"$or": [{"status": {"$eq": "open"}}, {"status": {"$eq": "pending"}}]}` |
| `$and` | AND compound | `{"$and": [{"status": {"$eq": "active"}}, {"amount": {"$gte": 100}}]}` |

### Sorting
Array of field strings, optional `:asc` / `:desc` (default `:asc`):
```json
["created_at:desc"]
["status:asc", "created_at:desc"]
```

### Pagination
```json
{ "limit": 20, "page": 1 }
```
Default `limit=20`, `page=1`. Max `limit=100`. Check `pagination.last_page` to detect multi-page results.

### Template-specific filters
`TemplateTool` uses `name_prefix` and `search` instead of Purity filters — see `templates.md`.

## Common Workflows

**Look up a client and their recent orders**
```
1. ClientTool list  {"filters": {"email": {"$contains": "acme"}}}
2. OrderTool list   {"filters": {"user_id": {"$eq": "<client_id>"}}, "sort": ["created_at:desc"], "limit": 5}
```

**Open tickets and their messages**
```
1. TicketTool list        {"filters": {"status": {"$eq": "open"}}, "sort": ["created_at:desc"]}
2. TicketTool show        {"action": "show", "ticket_number": "TKT-42"}
3. TicketMessageTool list {"action": "list", "ticket_number": "TKT-42", "sort": ["created_at:asc"]}
```

**Client subscription + unpaid invoices**
```
1. ClientTool show       {"action": "show", "client_id": "<id>"}
2. SubscriptionTool list {"filters": {"user_id": {"$eq": "<id>"}}}
3. InvoiceTool list      {"filters": {"user_id": {"$eq": "<id>"}, "status": {"$eq": "unpaid"}}}
```

**All overdue invoices**
```
InvoiceTool list {
  "filters": {"$and": [
    {"status": {"$eq": "unpaid"}},
    {"due_date": {"$lt": "<today_iso>"}}
  ]},
  "sort": ["due_date:asc"], "limit": 50
}
```

**Tasks for an order**
```
1. OrderTool show     {"action": "show", "order_number": "ORD-99"}
2. OrderTaskTool list {"action": "list", "order_number": "ORD-99"}
```

## Error Handling

| Error | Meaning | Action |
|---|---|---|
| `"not found"` | Record missing or no access | Verify id/number; `list` with a strict filter first |
| `"Unauthenticated"` | Token expired/invalid | Re-authenticate / refresh the OAuth session |
| `"Forbidden"` / permission error | Token's staff role lacks the scope | Enable that permission in Wayfront settings |
| Validation error | Required param missing or wrong type | Re-check the tool schema |
| Empty `data: []` | No matching records | Loosen filters or confirm data exists |

- If `show` returns not found, fall back to a strict `list` to confirm the record.
- If `pagination.last_page > 1`, iterate `page: 2`, `page: 3`, … to collect everything.
- For nested tools, verify the parent exists (parent `show`) before querying children.

## Rate Limiting & Efficiency

- No published hard limit, but avoid tight loops of hundreds of sequential calls.
- Use `limit: 100` and paginate rather than per-record fetches.
- For bulk needs ("all clients with unpaid invoices"), use one filtered `list` — don't fetch one by one.
- Fetch the parent once, then query children by the parent's id/number.

## Tips

- Order/ticket identifiers are **numbers** (`ORD-42`, `TKT-7`), not numeric ids.
- `TagTool` returns all tags at once — no filter needed; use the ids when filtering other tools.
- `TemplateTool` `is_modified: true` means the template was customised away from its default.
- `LogTool` / `UserActivityTool` are your debugging trail — what happened and when.
- Each tool enforces permission scopes on the token; a permission error means enable that scope in Wayfront settings.
