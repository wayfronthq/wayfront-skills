---
name: wayfront
description: >-
  Set up and operate a Wayfront workspace. Use for configuring services, forms,
  delivery checklists, payments, and email; working with clients, deals, contracts,
  orders, tickets, and billing; or drafting and previewing portal templates.
  Discover supported actions in the connected MCP tools or Wayfront CLI.
metadata:
  version: 2.0.1
  homepage: https://wayfront.com
---

# Wayfront Skill

## What Wayfront Is

Wayfront is client portal software for digital agencies selling repeatable productized services. Founded in 2014 as Service Provider Pro (SPP) and rebranded as Wayfront in 2026, the platform enables self-service ordering and automated order-to-delivery workflows.

Wayfront connects each purchase to the right delivery workflow and gives paid clients native ways to buy again, renew, prepay, refer, or resell. Client, order, project, billing, support, and communication data stays in one permissioned operating state for staff, automations, the Wayfront CLI, and MCP-connected AI clients. Each agency gets its own **workspace** (for example, `acme.wayfront.com`), and its clients use a branded **portal** for orders, tickets, invoices, and subscriptions.

This skill is how an AI agent drives a workspace through Wayfront's developer platform. There are several surfaces. **Read the reference file for the one you're using.** Each is self-contained.

| You want to... | Surface | Read |
|---|---|---|
| Query business data (and write where allowed) from an MCP-capable client (Claude, Cursor, and similar) | **MCP server** | [`reference/mcp.md`](reference/mcp.md) |
| Script the workspace from a terminal, or call any published API operation | **CLI / REST API** | [`reference/cli.md`](reference/cli.md) |
| Draft and preview portal changes or edit email templates | **MCP templates** | [`reference/templates.md`](reference/templates.md) |
| Review a prebuilt workspace or set up its first client order | **Workspace onboarding** | [Workspace onboarding](reference/workflows.md#workspace-onboarding) |
| Set up a sales process, delivery checklist, or transactional email | **Workflows** | [`reference/workflows.md`](reference/workflows.md) |
| Find a product capability and verify availability | **Features** | [`reference/features.md`](reference/features.md) |

## The Main Surfaces

- **MCP server** uses one URL, `https://mcp.wayfront.com/mcp`. Choose the workspace during OAuth sign-in. Follow the [MCP guide](reference/mcp.md#connection-setup) to connect and use `tools/list` to see supported actions.
- **CLI** (`npx wayfront`): resource-first commands (`wayfront orders list`), template sync (`wayfront templates pull/push`), and generated OpenAPI operations (`wayfront api call <operationId>`). The CLI reads the OpenAPI spec from `https://[WORKSPACE].wayfront.com/api.yaml` at runtime, so the operation list reflects the workspace's current generated spec.
- **REST API:** the same OpenAPI surface the CLI drives. OAuth sessions hit `/oauth-api/*`, static tokens hit `/api/*`. Discover operations with `wayfront api list`.
- **Templates** use Twig for portal and email views. Use a [portal draft](reference/templates.md#draft-and-preview-a-portal-change) for a requested portal design change. CLI template commands synchronize live overrides outside the generated OpenAPI operations.

All surfaces share the workspace's permission model: OAuth or token where supported gets you in, but staff **permissions** decide what each call can actually do.

## Core Data Model

```
Workspace
- Clients ............. the CRM record for a lead or customer
  - Deals ............ sales opportunities with an owner, stage, value, and conversation
  - Contracts ........ client-specific documents for review and signature
  - Signed agreements  signatures collected through forms, attached to the relevant record
  - Orders ........... a purchased service being delivered
    - OrderMessages .. the client/staff thread on an order
    - OrderTasks ..... checklist items (internal or client-facing)
  - Tickets .......... support requests
    - TicketMessages
  - Invoices ......... what's billed; one or more line items
  - Subscriptions .... recurring billing; renewals update invoices and follow the service's renewal behavior
  - Activities ....... CRM follow-ups and reminders on the client (internal)
  - Companies / client teams ... a client's invited coworkers, with per-area permissions
- Services ........... sellable offerings with pricing, attached forms, and reusable task templates
- Forms / FilledFormFields ... intake/onboarding/order/contact forms plus captured answers
- Coupons / credits / referrals ... discounts, account balance, affiliate payouts
- Pipelines / PipelineItems .. sales and production boards for deals and delivery records
- Templates .......... portal and email Twig templates
- Tags ............... internal labels on orders, tickets, clients (not client-facing)
- Team ............... staff/employee accounts (assignees)
- Logs / UserActivity  system and user audit trail
```

For orders and tickets, status is the single client-facing progress state. Client statuses are CRM lifecycle labels. Tags are internal-only labels.

### How entities relate

| From | Links to |
|---|---|
| Client | its deals, contracts, signed agreements, orders, tickets, invoices, and subscriptions |
| Deal | the client, a sales pipeline card, related invoices, and optionally a company and service |
| Contract | the client, optional service line items, and the invoice or orders it creates when configured |
| Order | the client, the service it came from, its invoice, its subscription, its messages and tasks, tags |
| Ticket | the client, the order it relates to, its messages, tags |
| Invoice | the client, the subscription, its line items (one per service line) |
| Subscription | the client, the service(s) it bills, invoices, and any renewal delivery work configured by those services |
| Service | the orders and subscriptions created from it, its intake and onboarding forms, and its reusable task templates |

Children hang off a client, so you scope a client's records by the client's user reference. The exact field and filter names live in the schema (see below), not here.

### Two conventions worth knowing

- **Identifiers:** most entities are addressed by a numeric `id`, but **orders and tickets use a string `number`** as their route key, not the numeric id. When you need an identifier you don't have, `list` first and read it off the result; the schema names the exact id parameter for each tool.
- **Status types vary by entity.** Order, ticket, and client filters use numeric codes that can vary by workspace. Deals use string statuses. Read the current values from the relevant tool schema.

## Discover the Surface at Runtime (do not memorize it)

This skill teaches concepts, not a frozen catalogue. Wayfront adds tools, fields, actions, and statuses continuously, so the authoritative list of what exists right now is always the live schema, never these docs. Before acting:

- **MCP:** read `tools/list`. Each tool's schema names its actions, its filterable/sortable fields, the id parameter, and the status code map for this workspace.
- **CLI / REST:** run `wayfront <resource> --help`, and `wayfront api list` (optionally by tag) to see every operation in the generated OpenAPI spec.

Treat any specific field, action, or status mentioned in the reference docs as illustrative. Confirm it live.

## Rules of Thumb

- **Discover, don't assume.** New entities, fields, actions, and statuses appear at runtime. Trust the live schema and `api list` over memory.
- **List before show.** When you don't have an id or number, `list` with a filter first. It surfaces the right identifier and confirms the record exists.
- **Confirm access.** Use `workspace-info` when the workspace, staff role, or available actions are unclear. Check the relevant module and record visibility when access is denied.
- **Many features must be enabled.** Capabilities such as helpdesk, credits, referrals, reseller, SEO reporting, time tracking, webhooks, API, MCP, and Slack may depend on workspace modules, integrations, plan access, or credentials. "Wayfront can do X" and "this workspace has X on" are different questions. See [`reference/features.md`](reference/features.md).
- **Batch with filters, don't loop.** Use a higher page limit plus pagination, or a single filtered `list`, instead of per-record calls.
