# Wayfront Feature Map

A catalogue of what a Wayfront workspace can do, so an AI can locate a capability, answer "can it do X", and recommend the right feature. This is a map, not a manual: it stays at the level of what exists and why, and leaves exact fields, actions, and settings to the live schema (`mcp.md`, `cli.md`) and the in-app screens. For how the core pieces connect, read `workflows.md` first.

## Feature Availability (read this first)

Many Wayfront capabilities are controlled by workspace settings, modules, or integrations. "Wayfront supports this" and "this workspace has this enabled" are different questions, so verify availability before recommending a feature or assuming it works in a workspace.

- **Modules** are optional product areas the agency turns on, such as helpdesk, referrals, credits, manual payments, whitelabel/templates, reseller, webhooks, API, MCP, SEO reporting, Slack, and time tracking. Several are plan-gated: API, Make, SEO reporting, Slack, helpdesk, whitelabel/template access, and webhooks on trial/free plans.
- **Integrations** connect outside systems: Stripe, PayPal, GA4, Google Tag Manager, Mailchimp, ActiveCampaign, Klaviyo, Agency Analytics, Looker/Data Studio, Google Drive, Zapier, Make, Slack, Xero, and others. A connected integration can still be limited by plan, missing credentials, or staff permissions.

## Sales & CRM

- **Clients as CRM records:** contact and address details, companies, signup parameters (acquisition source), account managers, private agency notes, enriched profile data, and full order/ticket/invoice/subscription history.
- **Custom CRM fields:** store extra client data, visible to staff only or surfaced in the portal.
- **Custom client statuses:** define your own client lifecycle labels.
- **Scheduled activities:** follow-ups and reminders on a client, assigned to a team member, surfaced in the to-do view.
- **Pipelines (kanban):** sales and production boards with custom colored stages; cards reference real records; per-pipeline automations (trigger to actions); owner and visibility controls.
- **Coupons:** discounts for promotions, with redemption limits and affiliate tracking.

## Catalogue & Ordering

- **Services:** the sellable offerings; one-time, recurring, trial, or setup-fee pricing; variants; add-on services; multiple orders per purchase; grouped service quantities; folders for organization; per-service currency; assigned team members.
- **Service requests:** request-based subscriptions, optionally capped by total per period or active-at-a-time (an upsell lever), with request limits reset on payment.
- **Forms:** order, intake, onboarding, and contact forms, with a drag-and-drop builder, conditional field rules, pre-filled fields via link parameters, custom URLs, multi-page flows, bulk/spreadsheet input, rich text, hidden fields, custom HTML, captcha, phone, date, and signature fields.
- **Cart & checkout:** clients browse services, add to cart, apply coupons, and pay; public order forms do the same from a shareable link, with embedded same-page payment and add-on/upsell flows.
- **Lead capture:** public contact, quote request, and lead generation forms can feed sales or support workflows; embeddable forms can be used on landing pages.
- **Orders:** the delivery unit; customizable client-facing statuses, due dates and overdue views, priority queue, scoped search/filtering, customizable list views, bulk actions, live updates, related-order linking, internal and client-facing tasks with sequential assignment.
- **Tasks:** reusable per-service checklists; internal or client-facing; client-assignable steps (e.g. approvals); a unified to-do across orders.

## Billing & Payments

- **Invoices:** automated from payments and manual; marked paid automatically by processor events; PDF downloads; shareable payment links; billing-email recipients; reminders; partial payments; credit/refund invoices; sequential or customer-based numbering; custom prefixes; seller/buyer and billing-address details; assignment, sharing, hiding, voiding.
- **Subscriptions:** recurring billing with configurable renewal behavior, scheduled starts, upgrades/downgrades with proration, charge-now, pause, cancel now vs. at period end, reactivation, self-service upgrade/cancel, and a cancellation-feedback form. See `workflows.md`.
- **Dunning:** a failed recurring payment is recorded and triggers a client email (a no-login link to fix the card or pay) and a team notification; the subscription moves to past due once the processor or the overdue-invoice job confirms it, and retries continue per the processor's settings.
- **Account balance:** clients prepay a balance in bulk and spend it on services.
- **Credits (module):** sell one-time or recurring credits clients redeem against services.
- **Account statement:** clients see their full billing history in the portal.
- **Taxes:** configurable taxes by country/region, taxable flags per service, EU VAT validation, and billing-country confirmation for VAT handling.
- **Multi-currency:** per-service currency for agencies charging in more than one.
- **Manual / other payment methods (module):** support check, wire, and similar via a custom self-service payment method.
- **Test mode:** test payment flows before opening checkout to clients.
- **Payment processors:** Stripe (cards, SEPA, ACH, bank debit, 3DS, saved cards, data import) and PayPal are the mainstays, with additional and region-specific options available.
- **Accounting:** Xero invoice sync.

## Support & Communication

- **Helpdesk / tickets (module):** support requests from the portal, contact forms, embedded support widgets/forms, and email; staff-only notes; merging; spam handling; sources.
- **Email-to-ticket:** incoming email creates tickets or appends replies to existing tickets and orders, including CC collaborators and forwarding.
- **Ticket snoozing & auto-close:** snooze to resurface later (shown as Open to clients), and auto-close inactive tickets.
- **Messaging:** threaded messages on orders and tickets; internal notes; client-facing replies; team mentions; emoji reactions; read receipts; scheduled and snoozed messages; clients reply by email without logging in; out-of-office auto-reply, message signatures, saved replies, and template variables.
- **Ratings:** clients rate resolved work (CSAT).
- **Notifications:** granular per-area email and in-app notifications for team and clients; followers on orders/tickets; push notifications via the installable app.

## Client Portal & Branding

- **Client portal:** dashboard, sidebar navigation, orders, tickets, invoices, files, and scoped portal search.
- **Branding:** logo, colors, company name, timezone, contact link.
- **Custom domain:** serve the portal from the agency's own subdomain, plus custom email sending/receiving domains.
- **Sidebar / menu editor:** add, remove, rearrange, and categorize portal menu items; link clients to order forms, custom pages, reports, external resources, or support paths.
- **Per-client sidebar links:** share a client-specific link in one user's portal, useful for private report dashboards, shared folders, briefs, or resources that should not appear for every client.
- **Custom pages:** add extra content pages to the portal.
- **Portal dashboard / onboarding builder:** build the client's landing page with editable blocks and task checklist items; use it for first-run onboarding, ongoing KPIs, embedded reports, videos, instructions, and links.
- **Client reporting:** publish external reports through reporting modules, dashboard blocks, menu links, or per-client sidebar links, depending on whether the report is global, reusable, or client-specific.
- **SEO reporting (module):** track domains, keywords, rankings, backlinks, and AI/LLM mentions, and surface them as client-facing SEO reports; can draw on the SE Ranking integration for rank data.
- **File management:** order and ticket deliverables, uploads, folders, bulk operations, sharing, large uploads, optional Google Drive.
- **Language / translation:** translate or reword client-facing text.
- **Installable app (PWA):** install on phone/desktop for push notifications and a full-screen experience, no app store.
- **Custom scripts:** add site-wide or conversion-specific scripts/pixels when integrations do not cover the tracking need.
- **Keyboard shortcuts** for fast navigation and order/ticket actions.

### Portal customization guidance

Start with the no-code controls before changing Twig templates:

- Use the **menu editor** to simplify navigation, expose order forms, and add links to the resources clients need most.
- Use **per-client sidebar links** when one client needs a private dashboard, shared folder, document, or external report in their portal.
- Use the **portal dashboard / onboarding builder** when clients need a guided first experience, ongoing KPIs, embedded reports, videos, instructions, or service-specific next steps.
- Use **custom pages** for reusable static content such as process docs, FAQs, policies, or resource hubs.
- Use **language / translation settings** to reword standard portal labels, buttons, headings, and system copy before editing templates.
- Reach for **Twig templates** only when the standard menu, page, dashboard, branding, and language controls cannot express the change; see [`templates.md`](templates.md).

## Team & Access

- **Team & roles:** invite staff/contractors and define roles with granular permissions, including hiding customer/billing data and controlling internal vs. external messaging access.
- **Assignment:** assign orders, services, and clients to team members.
- **Client teams:** clients invite coworkers into a company with per-area permissions (orders, tickets, invoices); per-record collaborators CC someone into a single order or ticket.
- **Login as client (impersonation):** view the portal as a specific client to troubleshoot.
- **Time tracking (module):** staff log time against orders, for internal cost and productivity tracking.

## Reporting & Exports

- **Operational reports:** order status, assigned orders, completed orders, completed tasks, overdue work, response times during business hours, ratings, and weekly overview emails.
- **Revenue reports:** revenue by billing cycle, best clients by lifetime value, best performing services, taxes collected, unused account balances, and coupon usage.
- **Data exports:** transactions CSV, client/lead lists, invoice PDF ZIPs, filled form data, revenue grouped by client, services, ratings, and commission exports.

## Scale & Growth

- **Referral / affiliate program (module):** referral links per client, referral-only coupons, click and sale tracking, configurable commission levels (one-time or recurring, per client), manual or automatic approval, commission reports and exports, payouts to account balance, and external payout tracking.
- **Whitelabel and reseller (separate modules):** whitelabel rebrands the experience; the reseller program lets other agencies resell the agency's services from their own branded workspace, with automatic fulfillment and conversation syncing. They're related but toggled independently.
- **Conversion tracking:** custom site-wide and conversion scripts/pixels for ad platforms, plus analytics events through GA4 or Google Tag Manager.
- **Coupons and campaigns:** one-time or recurring coupons, time-limited discounts, minimum cart amounts, affiliate attribution, and email marketing opt-in where required.

## Automation & Integrations

- **Automations:** trigger-to-action rules (including on pipelines) to move work forward without manual steps.
- **Webhooks (module):** notify external scripts of workspace events.
- **Zapier and Make:** connect Wayfront to thousands of other apps via triggers and actions.
- **Slack (integration, plan-gated):** push workspace notifications into Slack channels.
- **Email marketing:** Mailchimp, ActiveCampaign, and Klaviyo (Wayfront itself sends only transactional email; campaigns go through these).
- **Transactional email:** upcoming payment reminders, sent email logs, and sending/receiving from the agency's domain.
- **Analytics:** Google Analytics (GA4), Google Tag Manager, Looker Studio, Agency Analytics.
- **Storage:** Google Drive.

## Developer & AI Surfaces

- **REST API + OpenAPI spec (module):** for building on the workspace (see `cli.md`).
- **CLI** (`wayfront`) for terminal access to data and templates (see `cli.md`).
- **MCP server (module)** for AI clients to query the workspace (see `mcp.md`).
- **AI assistant:** a built-in assistant that looks up orders/tickets/invoices, helps with templates, searches the knowledge base, and guides setup.
- **Templates:** customize client-facing and email views in Twig (see `templates.md`).

## Data, Security & Compliance

- **Import / export:** import existing data to migrate in; export clients, leads, transactions, form data, ratings, services, revenue data, and invoices in bulk.
- **Authentication:** email/password, magic links, two-factor authentication, password resets, client sign-up.
- **Security:** isolated per-tenant databases, role permissions, two-factor authentication, session history, audit logs, and signup controls such as blocking free-email domains.
- **GDPR-related controls:** email opt-in, data export, and account deletion request flows.
