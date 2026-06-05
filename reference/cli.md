# Wayfront CLI

Drive a workspace from the terminal: resource-first commands, template sync, and generated OpenAPI REST operations. Repo: `wayfronthq/wayfront-cli`. Requires Node.js 18+.

## Install

```bash
npx wayfront            # run without installing
npm install -g wayfront # or install globally
```

## Authenticate

OAuth is the main path:
```bash
wayfront auth login acme      # opens the browser, signs in, stores a local OAuth session
wayfront auth status          # show active workspace and auth type
wayfront auth logout [acme]   # remove a saved session
```

Multiple workspaces are supported. Switch the active one:
```bash
wayfront workspace list
wayfront workspace use acme
```

Running bare `wayfront` prints connection status and help.

### Config & auth internals
- Config file: `~/.config/wayfront/config.json` (override the dir with `WAYFRONT_CLI_CONFIG_DIR`). Holds the `default` workspace plus per-workspace `{ url, auth }`.
- Two auth types: `oauth` (browser session) and `token` (static). The base path differs: OAuth requests hit `/oauth-api/*`, token requests hit `/api/*`.
- Workspace URL defaults to `https://<workspace>.wayfront.com` unless an explicit `url` is stored (supports self-hosted and custom domains).

## Resource Commands

Commands are organized by resource, like `gh`:
```
wayfront <resource> <action> [path-params] [key=value...] [--json '<body>']
```

The resources and the actions each one supports come from the live API, and the set grows over time. **Don't assume a resource has full CRUD.** Discover what's actually available:
```bash
wayfront --help                  # list resources
wayfront <resource> --help       # list that resource's actions
wayfront api list [Tag]          # every generated OpenAPI operation, optionally by tag
```

Examples (verify against `--help` for your workspace):
```bash
wayfront orders list
wayfront orders list page=2
wayfront orders get <order-number>
wayfront services create --json '{ ... }'
wayfront clients create --json '{"email":"jane@example.com","name":"Jane"}'
```

### Conventions
- **Path params are positional**, query params are `key=value` (repeat a key for arrays), request bodies use `--json '<json>'`. Without `--json`, `key=value` pairs become query params (or the body for endpoints that take one).
- **Identifiers:** most resources use a numeric `id`. **Orders and tickets use their string `number`** (the route key), not the numeric id. Nested resources take the parent's route key (the order or ticket number, or the client id for client activities).
- **Status is a numeric code, not a label.** Read the codes from the MCP tool schema (see `mcp.md`) or by inspecting `list` output. They are workspace-specific for orders, tickets, and clients.
- Run `wayfront <resource> --help` whenever you're unsure.

## Generated API Operations

The CLI fetches the OpenAPI spec from `https://<workspace>.wayfront.com/api.yaml` at runtime, so the operation list is current for that generated spec.

```bash
wayfront api list                 # generated operations: operationId  METHOD path  summary
wayfront api list Orders          # filter by OpenAPI tag
wayfront api call <operationId> [key=value...] --json '<body>'
```

`api call` maps `key=value` args to path and query params (path params fill `{placeholders}`, the rest become query) and sends `--json` as the body. Discover the `operationId` with `api list`, then call it. Prefer dedicated operations over generic record updates where they exist (some legacy catch-all update routes are slated for removal).

## Templates

`wayfront templates pull|push|reset [name]` syncs Twig templates to a local `./templates/` folder you can version-control. See [`templates.md`](templates.md). Template endpoints are a separate surface from the OpenAPI spec, so they may not appear in `wayfront api list`; use the `templates` commands.

## Prerequisite: the API module

The REST API (and therefore the CLI's data commands) lives behind the **API module**, which an agency enables in the workspace (Settings, Modules) and which is plan-gated. If the module is off, authentication and calls fail regardless of credentials. Template commands depend on template access instead. Connect as a staff account; the API is not a client surface.

## Permissions

OAuth or token gets you into the API. The workspace still decides what you can do, per the signed-in staff role's permissions. A `403` or permission error means that role needs the relevant scope enabled in Wayfront settings, not that the endpoint is missing.

## Stable Command Reference

These top-level commands don't change. For resource actions, use `--help` and `api list`.
```
wayfront                           Show connection status and help
wayfront auth login [workspace]    Sign in with OAuth
wayfront auth status               Show the active workspace and auth state
wayfront auth logout [workspace]   Remove a saved auth session
wayfront workspace list            List configured workspaces
wayfront workspace use <name>      Switch the active workspace
wayfront templates pull [name]     Pull templates, or one by name
wayfront templates push [name]     Push changed templates, or one by name
wayfront templates reset [name]    Reset modified templates to default
wayfront api list [tag]            List generated API operations from api.yaml
wayfront api call <operationId>    Call a generated API operation by operationId
```
