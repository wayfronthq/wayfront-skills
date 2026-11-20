# Wayfront Templates

Use the workspace MCP tools to read Twig templates, preview portal changes in a draft, and update approved email templates. Check `tools/list` for your client's available actions.

## Choose the right control

Use branding, menu, dashboard, and language settings for changes those controls support. Branding, dashboard, and menu writes save live. General workspace setup can proceed through those controls and the service and form tools. Use Twig when the owner asks for a specific portal design change, custom page, or email template change.

## Draft and Preview a Portal Change

1. On the latest portal design, call `portal-customization-read` with `list` to find editable drafts. Reuse the intended draft or call `portal-customization-write` with `create` when a new one is needed.
2. Read the live source with `template-read` and `show`. If the draft already has a staged version, read it with `portal-customization-read` and `show`. For a visual-only change, put scoped CSS in `portal.public.custom_scripts` and preserve unrelated scripts and styles. For a new page, use a `custom.` name.
3. Stage the full Twig source with `portal-customization-write` and `update_template`. Pass the current `draft_id`, `revision`, and `template_name`. Verify the change at `preview_url` or the custom page's direct preview link.
4. Share the preview with the intended reviewer. Anyone with the link can view it while valid using a sample client with actions disabled. The draft expires after seven days.
5. Use `publish` only after the user explicitly approves the live change. If the revision or live source changed, read the latest state before proceeding. Publish a custom page before adding a live menu link to it.

Use `undo` to revert an unpublished edit or `discard` to remove an unwanted draft. Both use `portal-customization-write`.

## Edit an Email Template

1. Confirm the workspace has a verified custom sending domain and template access. See [workspace email setup](workflows.md#workspace-email-setup).
2. Find the notification with `template-read` and read its full source with `show`. Keep the intended variables, links, and surrounding markup.
3. Apply an approved change with `template-write` and `update`. This replaces the full source and saves live. Use the notification's preview or test email in **Settings → Emails** to check rendering and links.

Use `template-write` with `reset` only when the user approves removing a live override. Shipped templates fall back to their defaults. Resetting a custom page deletes it.

## Find the Template Name

Use the name returned by `template-read`. List by prefix or search, then call `show` for the full Twig source. `is_modified` identifies a workspace override.

| Prefix | Renders |
|---|---|
| `portal.*` | Portal pages such as `portal.invoices.show` and shared files such as `portal.public.custom_scripts`. Some surfaces return these without the `portal.` prefix. |
| `clients.*` | Legacy client portal pages |
| `email.*` | Transactional emails |
| `payments.*`, `plugins.*` | Checkout and plugin views |
| `custom.*` | Custom pages |

Keep existing template variables intact and check the original source before using a variable on another page. Public pages and emails receive different variables from signed-in portal pages. Test the affected portal views in the draft before publishing.

For Git-based synchronization of live templates, see the [CLI reference](cli.md#templates).
