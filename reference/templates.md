# Wayfront Templates

Templates are the **Twig** files that render the client portal and outgoing emails. Each one has a filesystem **default** shipped with Wayfront, and a workspace can **override** any of them in the database. An override with custom content reports `is_modified: true`. Resetting a template deletes the override and falls back to the default.

## Naming

Templates use **dot-notation** that maps to folders, with a `.twig` extension:

```
invoices.show          ->  ./templates/invoices/show.twig
public.order_form      ->  ./templates/public/order_form.twig
email.client_receipt   ->  ./templates/email/client_receipt.twig
custom.welcome         ->  ./templates/custom/welcome.twig
```

Common namespaces (use `name_prefix` to scope a search):

| Prefix | Renders |
|---|---|
| `orders.*`, `invoices.*`, `subscriptions.*`, `tickets.*`, `team.*`, `public.*`, `layouts.*` | New portal pages exposed by the API/CLI. The underlying database override may store these with a `portal.` prefix. |
| `clients.*` | Legacy client portal pages |
| `email.*` | Transactional emails (e.g. `email.client_receipt`) |
| `payments.*`, `plugins.*` | Checkout/payment screens and plugin views |
| `custom.*` | Brand-new templates you author (see "Creating new templates") |

The authoritative list is whatever the workspace returns. Pull them or query `TemplateTool` rather than assuming.

## CLI Workflow (recommended for editing)

```bash
wayfront templates pull              # download all templates to ./templates/
wayfront templates pull invoices.show # or just one

# ...edit the .twig files locally, commit them to git...

wayfront templates push              # push only changed templates
wayfront templates push invoices.show
wayfront templates reset invoices.show # discard override, revert to default (then pull to refresh local copy)
```

- `./templates/` is yours to version-control. Pull once, edit over time, push when ready.
- `push` (no name) compares local files against the remote and uploads only the diffs. A brand-new file is created, an existing one is updated.
- `reset` (no name) lists every modified template and, on confirmation, reverts them all to default.
- Requires the `settings_templates` permission and template access on the workspace.

## Editing vs. creating

- **Editing a shipped template** (anything under `orders.*`, `invoices.*`, `email.*`, etc.): `pull`, edit, `push`. The push updates the DB override.
- **Creating a new template**: the name must start with `custom.` (e.g. `custom.welcome`). Creating a non-`custom.` name that has no shipped default is rejected by the API.
- **Resetting** only works on templates that have a DB override. Calling reset on a filesystem-only default (one that was never modified) returns a 404; there is nothing to revert.

## Read & write via MCP / API

- **MCP `TemplateTool`** is read-only: `list` (filter by name prefix like `"email."`/`"invoices."`, or search a name/content substring) and `show` by name. It surfaces both DB overrides and filesystem defaults; `is_modified` flags the customised ones. To write templates, use the CLI.
- The CLI wraps a small template REST surface (read, update, create, reset). These endpoints are separate from the public OpenAPI spec, so they won't show in `wayfront api list`. Use the `templates` commands rather than calling them by hand.

## Authoring Conventions

- **Twig `if`:** always write `{% if (condition) %}` with a space after `if`, never `{% if(condition) %}`.
- **New-design tenants** use the portal view set. The API/CLI usually expose these names without `portal.` (for example `invoices.show`), while some stored DB overrides include the `portal.` prefix. Prefer the new portal templates over legacy `clients.*` templates when both exist.
- Keep existing template variables intact. A template receives a fixed context from the renderer, so inventing variable names won't populate them. When unsure what's available, start from the pulled default and modify it rather than writing from scratch.
- On logged-in portal pages, `user` is available. Public pages and email templates only have the variables their renderer passes, so inspect the pulled default before assuming a variable exists.
- To inspect an object's available properties while developing, temporarily render it as JSON: `<pre>{{ user|json_encode|raw }}</pre>` (or `order`, `invoice`, `ticket`, etc.). Remove debug output before pushing.
- Don't add margin or spacing to containers that may render empty.
- Format and lint before pushing if your editor supports Twig or Prettier.

## Tips

- Diff first: `wayfront templates pull` into a clean git tree, edit, and review the diff before `push`.
- To experiment safely, copy the default, change one thing, push, verify in the portal, iterate.
- If a push fails with a permission error, the staff role behind your session lacks `settings_templates`.
