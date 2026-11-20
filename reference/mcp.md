# Wayfront MCP Server

Connect an AI agent over Streamable HTTP with OAuth 2.1. The agent queries and acts within the permissions of the staff account that authorizes the connection.

## Connection Setup

Enable the **MCP module** under **Settings → Modules**. Use **`https://mcp.wayfront.com/mcp`** with **Streamable HTTP**. In the browser, choose the workspace by name or domain, sign in with a staff account, and approve access.

### Claude Code

```bash
claude mcp add --transport http wayfront https://mcp.wayfront.com/mcp
```

In Claude Code, run `/mcp`, select Wayfront, and approve the connection in your browser.

### Claude and ChatGPT

Add the Wayfront connector in Claude or the Wayfront app in ChatGPT. Enter your workspace name or domain and approve access. The available tools and response fields can differ by client. See the [Wayfront connection instructions](https://wayfront.com/help/knowledgebase/mcp).

### When an approved app is missing a feature

If your agency needs a tool it doesn't see in an approved app, connect directly to `https://mcp.wayfront.com/mcp` from a compatible client such as Claude Code. Direct workspace MCP can expose a broader tool set. For example, it includes deal management and the workspace-wide follow-up queue, which the Wayfront app in ChatGPT omits. Check `tools/list` after connecting. Your workspace modules and staff permissions still apply.

## How the Tools Are Shaped

Read `tools/list` for the connected client's tools and actions.

- `order-read`, `ticket-read`, `client-read`, and `service-read` offer `list` and `show`. Changes use the corresponding `*-write` tool and its supported actions. Use `invoice-tool` and `subscription-tool` to read billing records.
- Nested tools for order messages, order tasks, and client activities take a parent identifier. Orders and tickets use their string number. Read the schema for the exact identifier field.
- Use `search-wayfront-docs` for product behavior or setup steps, then `get-wayfront-doc` for the relevant full document. Use the workspace tools to verify the workspace's records and configuration.

## Permissions and Access

The schema describes available actions. Permissions, record visibility, modules, and workspace state determine whether each call succeeds. Use `workspace-info` if the workspace, staff role, or capability is unclear. For a dedicated agent, authorize it with a staff account scoped to the work it needs to do.

If access is denied, check the workspace and role, then check the module and record scope before asking a workspace admin about access. An empty result means no visible record matched the query.

## Reading: Filter, Sort, Paginate

Most list actions accept `filters`, `sort`, `limit`, and `page`. Use the exact fields and operators in the tool schema. For example, to inspect services with “Monitoring” in the name, call `service-read` with:

```json
{
  "action": "list",
  "filters": {"name": {"$contains": "Monitoring"}},
  "limit": 20
}
```

Common filter operators include `$eq` and `$ne` for equality, `$in` and `$notIn` for membership, comparison operators such as `$gte`, and `$null` for missing values. The schema also identifies any supported compound filters.

Order, ticket, and client status codes can vary by workspace. Use the numeric values in each tool's filter schema. Deals use string statuses such as `open`, `won`, and `lost`.

Sort is an array such as `["created_at:desc"]`. Use the response's pagination and continue while `current_page < last_page` when the user asks for all matching records. Otherwise, report the scope or limit you used. Call `show` for records that need detail.

## Applying Changes

Verify the target and get approval for consequential changes. Messages and order changes can notify clients or trigger integrations. Use the exact action from the write tool's schema.

Preserve existing values in fields documented as full replacements. Re-read the affected records after a write. If the result is ambiguous or a step fails partway through, check what was saved before retrying. If the connection lacks the action, explain the limitation and give the relevant step in Wayfront.

### Create an order for a client

Use `order-read` with `create_options` to verify the selected client, service, request entitlement, and intake fields before calling `order-write` with `create`. If the request came from a helpdesk ticket, pass its `ticket_number` to link it. When a request entitlement is ineligible, tell the user why and get approval before considering a standalone order. Use client-provided answers for intake fields the client can see. If the create result is unclear, look up the order before retrying.

For service and form changes, follow the [workspace onboarding workflow](workflows.md#workspace-onboarding). For approved portal design changes, follow the [template draft workflow](templates.md#draft-and-preview-a-portal-change).

For sales opportunities, use `deal-read` and `deal-write` when they appear in the connected client's tool list. Create and send contracts from the client's profile in Wayfront. See [deals and contracts](workflows.md#deals-and-contracts).
