---
name: wayfront
version: 2.0.0
description: >-
  Work with a Wayfront workspace as an AI agent — three ways: the MCP server
  (query/update business data: clients, orders, tickets, invoices,
  subscriptions, services), the CLI (`wayfront`, resource commands + raw API +
  template sync), and template/service authoring. Schema-first: tools and API
  operations are discovered at runtime, so this skill stays correct as Wayfront
  adds endpoints.
install: clawhub install wayfront
metadata:
  openclaw:
    primaryEnv: WAYFRONT_MCP_TOKEN
    homepage: https://wayfront.com
---

# Wayfront Skill

## What Wayfront Is

Wayfront is a client portal and agency management platform built for productized digital agencies. It centralises sales, onboarding, project delivery, billing, and support into one platform. Each agency runs its own **workspace** (e.g. `acme.wayfront.com`); clients log in through a branded **portal** to view orders, tickets, invoices, and subscriptions.

This skill teaches AI agents the three ways to work with a workspace. **Read the reference file for the surface you're using** — each is self-contained.

| You want to… | Surface | Read |
|---|---|---|
| Query or update business data from an MCP client (Claude, ChatGPT, Cursor, OpenClaw) | **MCP server** | [`reference/mcp.md`](reference/mcp.md) |
| Script the workspace from a terminal, or call any API operation | **CLI / REST API** | [`reference/cli.md`](reference/cli.md) |
| Edit portal & email templates (Twig) | **Templates** | [`reference/templates.md`](reference/templates.md) |
| Create or configure services (the product catalogue) | **Services** | [`reference/services.md`](reference/services.md) |

## The Three Surfaces

- **MCP server** (`https://[WORKSPACE].wayfront.com/mcp`) — Streamable HTTP + OAuth 2.1. Best for conversational agents that read and act on data. Tools are entity-oriented (`OrderTool`, `TicketTool`, …) with `list`/`show` (and `update` where permitted). **Schema-first** — always inspect the live `tools/list` schema for the authoritative field list.
- **CLI** (`npx wayfront`) — resource-first commands (`wayfront orders list`), template sync (`wayfront templates pull/push`), and a raw escape hatch (`wayfront api call <operationId>`) over the full REST API. The CLI reads the OpenAPI spec from `https://[WORKSPACE].wayfront.com/api.yaml` at runtime, so every endpoint is reachable even if it isn't a named command yet.
- **REST API** — the same OpenAPI surface the CLI drives. OAuth sessions hit `/oauth-api/*`; static tokens hit `/api/*`. Discover operations with `wayfront api list`.

All three share the workspace's permission model: OAuth/token gets you in, but staff **permissions** decide what each call can actually do.

## Connecting

> Replace `[WORKSPACE]` with your workspace name — if you log in at `acme.wayfront.com`, your workspace is `acme`.

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
├── Clients
│   ├── Orders           — service delivery units (ORD-42)
│   │   ├── OrderMessages
│   │   └── OrderTasks
│   ├── Tickets          — support requests (TKT-7)
│   │   └── TicketMessages
│   ├── Invoices         — payment records
│   └── Subscriptions    — recurring billing plans
├── Services             — product/service catalogue (drives orders & subscriptions)
├── Forms / FilledFormFields — intake data attached to orders
├── Coupons              — discount codes
├── Templates            — portal + email Twig templates
├── Tags                 — labels on orders, tickets, clients
├── Team                 — staff/employee accounts
└── Logs                 — system activity audit trail
```

Identifiers: clients/invoices/services/subscriptions use numeric `id`; **orders and tickets use human numbers** (`ORD-42`, `TKT-7`), not their numeric id.

## Rules of Thumb

- **List before show.** When you don't have a specific id/number, `list` with a filter first — it surfaces the right identifier and confirms the record exists.
- **Schema-first, always.** New entities, fields, and API operations appear at runtime. Trust the live MCP schema / `api.yaml`, not memory.
- **Permissions gate everything.** A `Forbidden`/permission error means the token's staff role lacks that scope, not that the feature is missing.
- **Batch with filters, don't loop.** Use `limit: 100` + pagination or a single filtered `list` instead of per-record calls.
