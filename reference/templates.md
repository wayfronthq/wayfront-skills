# Wayfront Templates

Templates are the **Twig** files that render the client portal and outgoing emails. Each one has a filesystem **default** shipped with Wayfront; a workspace can **override** any of them in the database. An override with custom content reports `is_modified: true`. Resetting a template deletes the override and falls back to the default.

## Naming

Templates use **dot-notation** that maps to folders, with a `.twig` extension:

```
portal.invoices.show  →  templates/portal/invoices/show.twig
portal.home           →  templates/portal/home.twig
email.invoice_paid    →  templates/email/invoice_paid.twig
```

Common namespaces (use `name_prefix` to scope a search):

| Prefix | Renders |
|---|---|
| `portal.*` | Client-facing portal pages — orders, invoices, subscriptions, tickets, onboarding/contact forms, layouts, team, public pages |
| `email.*` | Transactional emails (e.g. `email.invoice_paid`) |
| `payments.*`, `public.*`, `plugins.*` | Checkout/payment screens, public pages, plugin views |

The authoritative list is whatever the workspace returns — pull them or query `TemplateTool` rather than assuming.

## CLI Workflow (recommended for editing)

```bash
wayfront templates pull            # download all templates to ./templates/
wayfront templates pull portal.home  # or just one

# ...edit the .twig files locally, commit them to git...

wayfront templates push           # push only changed templates
wayfront templates push portal.home
wayfront templates reset portal.home  # discard override, revert to default (then pull to refresh local copy)
```

- `./templates/` is yours to version-control — pull once, edit over time, push when ready.
- `push` (no name) compares local files against the remote and uploads only the diffs; a brand-new file is created, an existing one is updated.
- `reset` (no name) lists every modified template and, on confirmation, reverts them all to default.
- Requires the `settings_templates` permission and template access on the workspace.

## Read via MCP / API

- **MCP `TemplateTool`** — `list` (filter by `name_prefix` like `"email."`/`"portal."`, or `search` a name/content substring) and `show` (`template_name`). Shows DB overrides *and* filesystem defaults; `is_modified` flags customised ones.
- **REST API** — `GET /api/templates`, `GET /api/templates/{name}`, `PUT /api/templates/{name}` (`{ "data": "<twig>" }`), `POST /api/templates` (`{ "name", "data" }`), `DELETE /api/templates/{name}` (reset). These are exactly what the CLI calls.

## Authoring Conventions

- **Twig `if`:** always write `{% if (condition) %}` **with a space** after `if` — never `{% if(condition) %}`.
- **New-design tenants** are served from the `portal.*` namespace; when adding portal features, prefer `portal.` templates over the legacy set.
- Keep existing template variables intact — a template receives a fixed context from the renderer; inventing variable names won't populate them. When unsure what's available, start from the pulled default and modify it rather than writing from scratch.
- Don't add margin/spacing to containers that may render empty.
- Format/lint before pushing if your editor supports Twig/Prettier.

## Tips

- Diff first: `wayfront templates pull` into a clean git tree, edit, and review the diff before `push`.
- To experiment safely, copy the default, change one thing, push, verify in the portal, iterate.
- If a push fails with a permission error, the staff role behind your session lacks `settings_templates`.
