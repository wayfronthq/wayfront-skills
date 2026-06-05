# Wayfront CLI

Drive a workspace from the terminal: resource-first commands, template sync, and a raw escape hatch over the full REST API. Repo: `wayfronthq/wayfront-cli`. Requires Node.js 18+.

## Install

```bash
npx wayfront            # run without installing
npm install -g wayfront # or install globally
```

## Authenticate

OAuth is the main path:
```bash
wayfront auth login acme      # opens the browser, signs in, stores a local OAuth session
wayfront auth status          # show active workspace + auth type
wayfront auth logout [acme]   # remove a saved session
```

Multiple workspaces are supported; switch the active one:
```bash
wayfront workspace list
wayfront workspace use acme
```

Running bare `wayfront` prints connection status and help.

### Config & auth internals
- Config file: `~/.config/wayfront/config.json` (override dir with `WAYFRONT_CLI_CONFIG_DIR`). Holds `default` workspace + per-workspace `{ url, auth }`.
- Two auth types: `oauth` (browser session) and `token` (static). The base path differs:
  - OAuth → requests hit `/oauth-api/*`
  - Token → requests hit `/api/*`
- Workspace URL defaults to `https://<workspace>.wayfront.com` unless an explicit `url` is stored (supports self-hosted / custom domains).

## Resource Commands

Organized by resource, like `gh`. Pattern: `wayfront <resource> <action> [path-params] [key=value...] [--json '<body>']`.

Command groups: `auth`, `workspace`, `templates`, `clients`, `client-activities`, `coupons`, `filled-form-fields`, `invoices`, `logs`, `order-messages`, `order-tasks`, `orders`, `services`, `subscriptions`, `tags`, `tasks`, `team`, `ticket-messages`, `tickets`, plus `api` (raw fallback).

Most resources expose `list | get | create | update | delete`. Notable extras:

| Resource | Extra actions |
|---|---|
| `invoices` | `charge`, `mark-paid` |
| `tasks` | `complete`, `incomplete` |
| `subscriptions` | `list`, `get`, `update` only (no create/delete) |
| `team`, `tags`, `logs` | read-only |
| nested (`order-messages`, `order-tasks`, `ticket-messages`, `client-activities`) | take the **parent** id as the first positional arg |

```bash
wayfront orders list
wayfront orders list page=2
wayfront orders get ord_123
wayfront orders update ord_123 --json '{"status":"in_progress"}'
wayfront tickets create --json '{"subject":"Help","message":"Need a hand"}'
wayfront ticket-messages list ticket_123 limit=50
wayfront invoices charge inv_123 --json '{"payment_method":"manual"}'
wayfront clients create --json '{"email":"jane@example.com","name":"Jane"}'
```

Conventions:
- **Path params are positional** (`wayfront orders get ord_123`).
- **Query params are `key=value`** (`page=2`, `limit=50`); repeat a key for arrays.
- **Request bodies use `--json '<json>'`.** Without `--json`, `key=value` pairs become query params (or the body for endpoints that take one).
- Run `wayfront <resource> --help` for that resource's actions.

## Raw API Escape Hatch

Anything not yet a named command — and any newly added endpoint — is reachable raw. The CLI fetches the OpenAPI spec from `https://<workspace>.wayfront.com/api.yaml` at runtime, so the operation list is always current.

```bash
wayfront api list                 # all operations (operationId  METHOD path - summary)
wayfront api list Orders          # filter by OpenAPI tag
wayfront api call ordersUpdate order=ord_123 --json '{"status":"paid"}'
```

`api call <operationId>` maps `key=value` args to path/query params (path params fill `{placeholders}`; the rest become query params) and sends `--json` as the body. This is the most future-proof way to hit the API — discover the `operationId` with `api list`, then call it.

## Templates

`wayfront templates pull|push|reset [name]` syncs Twig templates to a local `./templates/` folder you can version-control. See [`templates.md`](templates.md) for the full workflow, naming, and authoring conventions.

## Permissions

OAuth/token gets you into the API; the workspace still decides what you can do:
- Workspaces without template access can't pull/push templates.
- Staff without `settings_templates` can't reach template endpoints.
- Other endpoints work through OAuth when the workspace allows them.

A `403`/permission error means the staff role behind your session needs that scope enabled in Wayfront settings — not that the endpoint is missing.

## Command Reference

```
wayfront                                 Show connection status and help
wayfront auth login [workspace]          Sign in with OAuth
wayfront auth status                     Show the active workspace and auth state
wayfront auth logout [workspace]         Remove a saved auth session
wayfront workspace list                  List configured workspaces
wayfront workspace use <workspace>       Switch the active workspace
wayfront templates pull [name]           Pull templates, or one by name
wayfront templates push [name]           Push changed templates, or one by name
wayfront templates reset [name]          Reset modified templates to default
wayfront <resource> list|get|create|update|delete [...]
wayfront invoices charge|mark-paid <invoice> [--json ...]
wayfront tasks complete|incomplete <task>
wayfront api list [tag]                  List raw API operations from api.yaml
wayfront api call <operationId> [params] Call any raw API operation by operationId
```
