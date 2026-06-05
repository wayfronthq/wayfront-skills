---
name: wayfront
version: 2.0.0
description: >-
  Work with a Wayfront workspace as an AI agent: the MCP server
  (query clients, orders, tickets, invoices, subscriptions, and services, and
  write where a tool exposes it), the CLI (`wayfront`, resource commands plus
  generated API operations and template sync), and template editing plus service/workflow setup. Schema-first:
  tools and API operations are discovered at runtime, so this skill stays correct
  as Wayfront adds endpoints.
metadata:
  homepage: https://wayfront.com
---

# Wayfront Skill

## What Wayfront Is

Wayfront is the AI-ready operating system for productized agencies: client billing, onboarding, and project delivery in one platform instead of five. It's built for agencies that sell services, not hours, and it runs the whole lifecycle (sell, deliver, scale). Each agency gets its own **workspace** (e.g. `acme.wayfront.com`), and their clients log in to a branded **portal** for orders, tickets, invoices, and subscriptions.

This skill is how an AI agent drives a workspace through Wayfront's developer platform. There are several surfaces. **Read the reference file for the one you're using.** Each is self-contained.

| You want to... | Surface | Read |
|---|---|---|
| Query business data (and write where allowed) from an MCP-capable client (Claude, Cursor, and similar) | **MCP server** | [`reference/mcp.md`](reference/mcp.md) |
| Script the workspace from a terminal, or call any published API operation | **CLI / REST API** | [`reference/cli.md`](reference/cli.md) |
| Edit portal and email templates (Twig) | **Templates** | [`reference/templates.md`](reference/templates.md) |
| Understand how services, forms, orders, and billing fit together (and set them up) | **Workflows** | [`reference/workflows.md`](reference/workflows.md) |
| Look up what a workspace can do (the full feature set) | **Features** | [`reference/features.md`](reference/features.md) |

## The Main Surfaces

- **MCP server** (`https://[WORKSPACE].wayfront.com/mcp`): Streamable HTTP plus OAuth 2.1. A real action surface for AI agents to query and act on a workspace. Tools are entity-oriented (`OrderTool`, `TicketTool`, and so on); many are `list`/`show`, a growing set also write. Staff-only (clients are refused) and gated by the MCP module; it acts with the permissions of the staff account it connects as, so scope an agent with a dedicated role. **Schema-first:** always inspect the live `tools/list` schema for the authoritative actions and fields.
- **CLI** (`npx wayfront`): resource-first commands (`wayfront orders list`), template sync (`wayfront templates pull/push`), and generated OpenAPI operations (`wayfront api call <operationId>`). The CLI reads the OpenAPI spec from `https://[WORKSPACE].wayfront.com/api.yaml` at runtime, so the operation list reflects the workspace's current generated spec.
- **REST API:** the same OpenAPI surface the CLI drives. OAuth sessions hit `/oauth-api/*`, static tokens hit `/api/*`. Discover operations with `wayfront api list`.
- **Templates:** a separate Twig template surface for portal and email views. Use `wayfront templates ...` rather than `api call`, because template endpoints are not part of the generated OpenAPI operation list.

All surfaces share the workspace's permission model: OAuth or token where supported gets you in, but staff **permissions** decide what each call can actually do.

## Connecting

> Replace `[WORKSPACE]` with your workspace name. If you log in at `acme.wayfront.com`, your workspace is `acme`.

**MCP (Claude Code):**
```bash
claude mcp add --transport http wayfront https://[WORKSPACE].wayfront.com/mcp
```
Authorization happens in your browser on first use. Other clients: see [`reference/mcp.md`](reference/mcp.md).

**CLI:**
```bash
npm install -g wayfront        # or: npx wayfront
wayfront auth login [WORKSPACE]
```
Full setup, config location, and resource commands: [`reference/cli.md`](reference/cli.md).

## Core Data Model

```
Workspace
- Clients ............. the customers (users); the CRM record; own everything below
  - Orders ........... a purchased service being delivered
    - OrderMessages .. the client/staff thread on an order
    - OrderTasks ..... checklist items (internal or client-facing)
  - Tickets .......... support requests
    - TicketMessages
  - Invoices ......... what's billed; one or more line items
  - Subscriptions .... recurring billing; renewals update invoices and follow the service's renewal behavior
  - Activities ....... CRM follow-ups and reminders on the client (internal)
  - Companies / client teams ... a client's invited coworkers, with per-area permissions
- Services ........... product catalogue; buying one creates an order/subscription
- Forms / FilledFormFields ... intake/onboarding/order/contact forms plus captured answers
- Coupons / credits / referrals ... discounts, account balance, affiliate payouts
- Pipelines / PipelineItems .. sales and production boards (cards reference orders, etc.)
- Templates .......... portal and email Twig templates
- Tags ............... internal labels on orders, tickets, clients (not client-facing)
- Team ............... staff/employee accounts (assignees)
- Logs / UserActivity  system and user audit trail
```

For orders and tickets, status is the single client-facing progress state. Client statuses are CRM lifecycle labels. Tags are internal-only labels.

### How entities relate

| From | Links to |
|---|---|
| Client | its orders, tickets, invoices, subscriptions |
| Order | the client, the service it came from, its invoice, its subscription, its messages and tasks, tags |
| Ticket | the client, the order it relates to, its messages, tags |
| Invoice | the client, the subscription, its line items (one per service line) |
| Subscription | the client, the service(s) it bills, invoices, and any renewal delivery work configured by those services |
| Service | the orders/subscriptions created from it, its intake form |

Children hang off a client, so you scope a client's records by the client's user reference. The exact field and filter names live in the schema (see below), not here.

### Two conventions worth knowing

- **Identifiers:** most entities are addressed by a numeric `id`, but **orders and tickets use a string `number`** as their route key, not the numeric id. When you need an identifier you don't have, `list` first and read it off the result; the schema names the exact id parameter for each tool.
- **Status is a number, not a label.** Statuses are stored as integer codes, and several (orders, tickets, clients) are customizable per workspace. Filter by the code, and read the current code-to-label map from the tool's live description. Never hard-code a status name.

## Discover the Surface at Runtime (do not memorize it)

This skill teaches concepts, not a frozen catalogue. Wayfront adds tools, fields, actions, and statuses continuously, so the authoritative list of what exists right now is always the live schema, never these docs. Before acting:

- **MCP:** read `tools/list`. Each tool's schema names its actions, its filterable/sortable fields, the id parameter, and the status code map for this workspace.
- **CLI / REST:** run `wayfront <resource> --help`, and `wayfront api list` (optionally by tag) to see every operation in the generated OpenAPI spec.

Treat any specific field, action, or status mentioned in the reference docs as illustrative. Confirm it live.

## Rules of Thumb

- **Discover, don't assume.** New entities, fields, actions, and statuses appear at runtime. Trust the live schema and `api list` over memory.
- **List before show.** When you don't have an id or number, `list` with a filter first. It surfaces the right identifier and confirms the record exists.
- **Status is a number.** Filter by the numeric code and read the per-workspace map from the tool schema.
- **Permissions gate everything.** A permission error means the staff role lacks that scope, not that the feature is missing.
- **Many features must be enabled.** Capabilities such as helpdesk, credits, referrals, reseller, SEO reporting, time tracking, webhooks, API, MCP, and Slack may depend on workspace modules, integrations, plan access, or credentials. "Wayfront can do X" and "this workspace has X on" are different questions. See [`reference/features.md`](reference/features.md).
- **Batch with filters, don't loop.** Use a higher page limit plus pagination, or a single filtered `list`, instead of per-record calls.
