# Wayfront Workflows: How It Fits Together

This is the mental model an AI needs to set a workspace up well and suggest improvements. It's conceptual on purpose: the exact fields, actions, and statuses live in the live schema (see `mcp.md` and `cli.md`), but the way the pieces fit together is stable.

Wayfront automates the agency lifecycle, sell then deliver then scale, around a few primitives that feed each other. For a flat catalogue of every capability (billing, support, portal, integrations, and so on), see [`features.md`](features.md).

## Workspace Onboarding

Help the owner get their primary service ready for a client to buy and the team to deliver. Ask one or two questions per turn and use the owner's wording for their services.

### Inspect the workspace first

Discover the available actions in [`tools/list`](mcp.md#how-the-tools-are-shaped). Confirm the workspace with `workspace-info`. Inspect the catalog, checkout, and attached forms with `service-read` and `form-read`. If access is limited, explain what you could verify and ask the owner for the missing context.

- **Built from the owner's website or already configured:** Start by reviewing the primary service and its existing forms. Preserve usable services and forms and propose targeted edits.
- **Sample catalog:** Ask what they sell if the sample offerings do not match their business. Keep any offerings the owner confirms are real. If you are unsure whether the catalog is sample data, ask before proposing replacements.
- **Empty catalog:** Start with one question about what they sell. Get the billing model and price before proposing the setup. If they have yet to set a price, label any suggested price as a placeholder and get approval before saving it.

### Review one service package with the owner

Choose the buying path that matches how the owner sells. Use an order form for a package clients can buy at a known price. For quote-first work, use a contact form to collect the inquiry and send the invoice once the scope and price are agreed. Create and send manual invoices in Wayfront.

Summarize the service, price, and buying path in plain language. Walk through what happens at each step.

- **At checkout:** Keep it to name, email, service selection, and payment for self-service purchases. Suggest an optional “How did you hear about us?” question if attribution is missing.
- **For each project:** Use an intake form for the delivery brief, such as goals, pages, or deadlines. Aim for three to six questions and identify any optional extras.
- **Once per client:** Use an onboarding form for company-level details such as brand assets, default preferences, and requests for account access. Wayfront keeps these answers on the client profile for reuse across that client's orders. Move reusable questions out of project intake.
- **On each renewal:** Recommend a new order for an independent deliverable each cycle or reopening the same order for a continuing engagement. Where the service sells on-demand requests or only renews billing, review those [renewal options](#services-the-decision-guide) with the owner. Give the reason in one sentence and skip this decision for one-time services.
- **During delivery:** Put repeatable steps on the service checklist. Agree who owns each step and which steps need client participation, such as approving a deliverable. Add pipelines, notifications, and automations when they solve a specific handoff the owner already needs.

Review the primary service first. Agree with the owner which other services should follow the same pattern. Ask “Does this setup make sense, or would you like to make changes?” Apply only the approved proposal. If the owner requests changes to the proposal, revise it and ask again before writing.

A monthly SEO package with a fresh target list and deliverable each cycle gets a new order. Ask for target pages and priorities on each order, and request analytics access and brand assets once. A website care plan with one continuing backlog reopens the same order so the team keeps the conversation and history together.

### Apply and verify the approved setup

- **New service:** Reuse suitable forms or create the agreed intake and onboarding forms with `form-write`. Use `service-write` to create the service with the approved billing and renewal behavior and attach those forms. Add the returned service ID to the appropriate order form if clients will buy through checkout.
- **Existing service:** Update the verified service and form IDs. If a form update supplies `fields`, send the complete intended list and retain the IDs of every field you are keeping. Omitted fields are removed. Preserve existing service associations, including the full onboarding form list when changing it. Check active subscriptions before changing renewal behavior because it affects their future renewals.
- **Verification:** Re-read each changed service and its forms and verify the links. If a multi-step setup fails, inspect what was saved before retrying. If an action is unavailable, give the owner the corresponding step in Wayfront.

Help the owner finish one step at a time. Guide them through connecting Stripe in Wayfront if needed. Verify branding with `settings-read` and apply any approved changes through `settings-write`. If sending from their own domain, follow [workspace email setup](#workspace-email-setup). Before switching to live mode, have them run the chosen buying path in test mode. Check that it produces the expected order, that the client sees the right form, and that staff can see and act on the first delivery step. Keep portal template work for a separately requested, specific design change.

## The Core Loop

```
Service  ->  Invoice or Order Form  ->  payment  ->  Order ("project")  ->  Intake form  ->  Tasks  ->  Complete
   |                                       |                                                          |
   recurring? -> Subscription ------------ + ---- renewals invoice and follow service renewal behavior ----+
```

- A paid invoice creates one or more orders. Paying also auto-creates the client's portal account if they don't have one. You rarely create orders by hand.
- A service is a predefined offering plus its delivery behavior. Reuse it in invoices and order forms so every resulting order behaves the same.
- An order is the unit of work ("project"): status, messages, tasks, due date, intake data.
- A subscription is recurring billing. Each successful renewal records the payment, updates the invoice, and creates or reopens delivery work according to the service's renewal behavior, with no manual billing step.

So get services right and most of the rest (orders, billing, delivery) falls out automatically.

## Clients, Teams & CRM

- **A client is the CRM record.** Beyond login, a client carries contact and address details, signup parameters (where they came from), a private agency note, custom fields, and the full history of their orders, tickets, invoices, and subscriptions. That history is the CRM.
- **Activities** are follow-ups and reminders pinned to a client: a check-in call, an upsell nudge, a "circle back after the report" note. They can be scheduled and assigned to a team member, and they surface in a team to-do view. They are internal, clients don't see them.
- **Client teams.** A client can create a Company and invite team members (a coworker, the accounting department). Each member gets granular permissions: collaborate on orders, collaborate on tickets, and/or access invoices. Members sign in by magic link, can submit intake forms and create requests, but cannot manage payment details, and cannot start a team of their own. Keep a subscription on one account (e.g. billing) since payment management isn't shared.
- **Per-record collaborators.** Independently of full team membership, a client can CC a coworker into a single order or ticket. That person gets updates on just that record. Subscriptions, orders, and tickets each have their own collaborators.

### Pipelines (sales and production)
Pipelines are kanban-style boards with custom, colored stages. There are two kinds:
- **Sales pipelines** track deals and leads toward a close.
- **Production pipelines** track delivery work in flight.

A pipeline card points at a real record (commonly an order), so moving a card across stages is a view onto the same data the rest of the system uses, not a separate copy. Pipelines have an owner, a visibility setting, and an optional default. Use them when an agency wants a visual board on top of the order/CRM data.

Pipelines can carry **automations**: a trigger event (something happening to a card or its record) fires one or more actions. The available triggers and actions depend on the pipeline type. This is how a board can move work forward on its own rather than purely by hand.

### Deals and Contracts

- **Track a sales opportunity as a deal.** One client can have multiple deals. Use a sales pipeline stage and an owner for each opportunity, log the next follow-up, and keep related messages on the deal. Use **Clients → Sales Inbox** to catch replies awaiting the team. A staff member can create an invoice from the deal when the client is ready. See the [deals guide](https://wayfront.com/help/knowledgebase/deals).
- **Use a contract when the client should sign before billing or delivery.** From the client profile, use **Actions → Add contract**. Start from a reusable contract template when the terms recur, review the client's terms and any agreed service prices, then send the contract for signature. Sending freezes what the client will sign. Use the form's signature field for standard terms collected during checkout or intake instead.
- **Choose the action after signing.** Save the signed contract alone when there is no purchase yet. Create an unpaid invoice when the client should pay next or the service is recurring. Create orders directly only for one-time work that should start on signature without an invoice or subscription.

## Services: the Decision Guide

A service captures how something is sold and delivered. The choices that shape everything downstream:

- **Variants vs separate services.** Use service variants when the client is buying the same service workflow with the same intake form, tasks, renewal behavior, and fulfillment, but picks a different tier/size/option and price. Use separate services when tiers need different forms, task lists, assignees, renewal behavior, reporting, currencies, or clean standalone revenue tracking.
- **Billing.** One-time, recurring (with an optional free or paid trial), or free.
- **Renewal behavior** (recurring only): what happens each billing cycle. Pick by how the work repeats:
  - **Reopen the same order:** ongoing work on one thing (SEO for one site). Mark it complete each month, it returns to your queue next cycle.
  - **New order each cycle:** each cycle is independent work (one article per month, each its own project).
  - **Service requests:** the client subscribes and requests tasks on demand (up to N design requests per month). Each request is its own order.
  - **No action:** nothing to deliver on renewal (hosting).
- **Intake form.** Attach one if the team needs project details before starting. The order then waits in its initial status until the client fills it.
- **Tasks.** Define the [delivery checklist](#tasks-and-sops) on the service so new orders inherit it.
- **Variants.** A variant stores selectable option values and a variant price on the service. Clients choose variants from the portal or an order form; services with variants do not have a direct one-click payment link.

Inspect the existing service with `service-read` and check the current `service-write` schema before applying the approved choices.

### Service requests are an upsell lever
Request-based services can be unlimited or capped two ways: a total per billing period, or a number active at a time. The active-at-a-time cap is the better upsell mechanic, because higher tiers allow more concurrent work. When a client hits the cap, the New request action shows a limit-reached notice.

## Forms: Four Types, Four Jobs

Picking the right form for each job is what makes the client experience feel effortless.

| Form | When it runs | Scope | Produces |
|---|---|---|---|
| **Order form** | client selects services and pays | per checkout | an invoice, then order(s) |
| **Intake form** | after payment, inside the order | per order | project details on that order |
| **Onboarding form** | once per client | per client | static brand info reused on every order |
| **Contact form** | outside the order flow | standalone | a support ticket |

Key rules:
- **Keep checkout lean.** Don't collect project details on the order form, that lowers completion. Use an intake form (per order) instead. Payment stays fast, details come after.
- **Don't re-ask static info.** Brand assets, logos, and style guides belong in an onboarding form (per client), assigned to services so Wayfront shows it only if the client hasn't completed it. **Onboarding data is editable later:** the client can update it from their dashboard when their brand changes, and the agency can edit the stored fields from the client profile. Changes apply going forward without resubmitting.
- **Order forms** combine service-selection fields (single choice, add-ons, repeatable variants), account fields (email is the anchor), an optional embedded payment step, and a few data fields. More fields means lower completion.
- **Field rules** (show or hide a field based on another answer) work across all four form types.

If you build forms programmatically, the schema enforces the real requirements (for example an order form needs a service-bearing field and an email field). Read it, and start from an existing form's shape rather than from scratch.

Most agencies start with an order form plus an intake form, then add onboarding and contact forms as the workflow matures.

## Order Lifecycle & Delivery

Orders move through statuses, and **statuses are client-facing**: the client sees the current status of their order in the portal, so the names communicate progress to them. Defaults run from "waiting on the client" to "ready to work" to "in progress" to "complete", with a canceled state for refunds. Agencies can rename and add statuses to match their workflow.

- **Starting status depends on the service:** with an intake form, the order waits until the client submits it; without one, it can start ready-to-work (or skip straight to in-progress, a per-workspace setting).
- **Due dates** count down from when work can begin (waiting on the client doesn't burn time) and stop at completion. They only appear when the service has a delivery deadline.
- **Completion handling** is configurable: auto-reopen on a client reply, lock until the client requests a revision, or stay completed. Lock plus auto-reopen together means only orders that truly need input come back.
- **Tasks** advance the delivery checklist. Use a client task when the client needs to take an action in the portal.

**Tags vs statuses.** Don't confuse the two. For orders and tickets, status is the single client-facing progress state. Client status is a CRM lifecycle label. **Tags are internal labels** (agency only, many per record) for filtering and organizing orders, tickets, and clients. Clients never see tags.

### Tasks and SOPs

Turn the service's standard operating procedure into a reusable checklist. Start with the primary service and add a task for each handoff or checkpoint. Give each task a clear outcome, an owner, and a deadline where helpful. Put short instructions or a link to the SOP in the task description. Keep internal work and quality checks staff-only. Use a client-assigned task when the client needs to approve or provide something. For image feedback and revisions, use [file proofing](https://wayfront.com/help/knowledgebase/file-proofing) on the order.

For example, a content service could run through brief review, drafting, internal quality check, client approval, and delivery. Use **Orders → Tasks** to see the work queue and **Clients → Follow-ups** for sales or relationship reminders. See [Wayfront's guide to documenting delivery SOPs](https://wayfront.com/blog/scaling-productized-services-framework).

Use `service-read` with `get_task_templates` to inspect the checklist. `service-write` with `sync_task_templates` replaces the full list, so retain the IDs of the steps you are keeping and pass task deadlines in hours. Changes apply to new orders. A reopened order keeps its existing checklist and resets task completion. For an existing order, use `order-task-read` and `order-task-write` with its order number. Include `task_id` when completing or reopening a single task because omitting it applies the action to every eligible task on the order.

## Subscriptions

- **Renewal is automatic:** record the payment, update the invoice, apply the service's renewal behavior, and send confirmations.
- **Lifecycle:** active, past due (a payment failed and is being retried, back to active on success), paused, pending cancellation (cancels at period end, still reactivatable), and canceled (terminal).
- **Admin management** includes upgrade or downgrade (per item, optional proration), charge now (resets limits and shifts the billing date), pause, cancel now vs. cancel at period end, and reactivate a pending cancellation. Which actions apply depends on the payment processor (e.g. pause is Stripe-only; PayPal cancels/suspends from PayPal's side, and Wayfront syncs the status).

### Self-service upgrade and cancellation
Off by default, because whether clients should self-serve depends on the agency's commitments. Turn it on in payment settings to show upgrade and cancel links in the portal. Clients can then upgrade/downgrade or cancel their own Stripe subscription without contacting the agency.

**Collecting cancellation feedback:** optionally show a cancellation form when a client cancels, to capture why they're leaving. Under the hood it's a normal contact form, so the response lands as a ticket/submission you can review; after the client submits it, they're redirected to complete the cancellation. So you get the feedback and the client still cancels in one flow.

### Failed payments (dunning)
When a recurring charge fails, Wayfront records the failure, emails the client, and notifies the team; the subscription moves to past due once the processor status (or the overdue-invoice job) confirms it. The client's email carries a no-login link: for Stripe, to update the card or pay the outstanding balance directly (then Stripe retries on its schedule); for PayPal, to fix the payment method in PayPal. A fresh email goes out on each failed retry. When payment succeeds the subscription returns to active and the service's renewal behavior runs. If retries are exhausted, the processor's configuration decides whether it cancels.

## Communication

- **Order and ticket threads** keep the conversation attached to the work. Order messages can be team-only (internal notes) or client-visible. Tickets are the support channel, fed by contact forms and by email-to-ticket.
- **Saved replies** (canned snippets) speed up repetitive responses, and messages can be scheduled.
- **Ratings** let clients rate resolved work, giving the agency a CSAT signal.
- **Followers and notifications** decide who hears about what: staff follow orders they're assigned to (or all new orders), and clients and collaborators get notified on the events that involve them.

### Workspace Email Setup

Start in **Settings → Emails**. Set the sender display name and review the transactional emails clients will receive, such as sign-in links, order and intake updates, invoice notices, and contract emails.

1. **Sending from your own address.** Set up the custom workspace domain first. In email settings, add the sender address and publish the DNS records shown in Wayfront. Wait for verification before expecting messages to use the custom sender. MCP `settings-write` can change the sender display name. Configure the sending address and domain in Wayfront and the DNS provider. See the [workspace domain guide](https://wayfront.com/help/knowledgebase/setup#custom-email-sending-domain).
2. **Receiving replies.** Forward the custom inbox to the workspace forwarding address shown in email settings. Use a test client and order or ticket to generate a real message notification. Reply from the test client's inbox and check that the reply lands in the original conversation. If the workspace uses Helpdesk, also verify that a new email from the test inbox creates a ticket. See the [email-to-ticket guide](https://wayfront.com/help/knowledgebase/email-to-ticket).
3. **Editing notifications.** Preview or send a test email from Settings to check the sender, subject, sample variables, and link placement. Use the test client's real notification to verify where a link goes. For Twig changes, follow the [email template workflow](templates.md#edit-an-email-template). See the [email settings guide](https://wayfront.com/help/knowledgebase/emails).

## Scale

The growth channels are built into the portal, so an AI auditing a workspace can point agencies at them:
- **Referrals / affiliates:** when the referral module is enabled, approved affiliates can get referral links, with configurable commissions (one-time or recurring, per client, paid out to account balance or otherwise).
- **White-label reselling:** resellers sell the agency's services under their own branded portal and domain, with fulfillment and conversation syncing handled automatically.
- **Coupons** for promotions and tracking, and **account balance / credits** for refunds, referral payouts, or prepaid work.

## Optimizations an AI Can Suggest

When auditing a workspace, look for these patterns and recommend the fix. Each maps a symptom to a feature, grouped by lifecycle stage. Confirm the workspace's current setup (and plan) before recommending; not every feature is on by default.

### Sell (convert more, at higher value)
- **Heavy order forms** collecting project data: move it to an intake form so checkout stays short and completion rises.
- **A password field on the order form:** drop it and let clients sign in by magic link to cut signup friction.
- **Marketing pages linking to a generic order form:** use pre-filled links so the right service is preselected.
- **Same workflow sold in multiple tiers:** use service variants when only the selected tier/size and price change; split into separate services when delivery, forms, tasks, renewal behavior, or reporting should differ.
- **No add-ons at checkout:** offer add-on services to lift average order value.
- **No conversion tracking:** connect analytics and conversion pixels so order-form performance is measurable. Use GA4 **or** Google Tag Manager, not both: each emits its own purchase/e-commerce events, so running both double-counts conversions.
- **No promotions or referral tracking:** use coupons for campaigns and affiliate-tagged tracking.

### Deliver (less manual work, fewer round-trips)
- **Project data re-asked every order:** move static brand info to an onboarding form (per client).
- **Orders sitting in the initial status with no intake form:** let them start ready-to-work so clients aren't stuck.
- **Manual reassignment between steps:** define service tasks so assignment advances automatically as each step completes.
- **Approvals chased over email:** use client-facing/assigned tasks so the client acts inside the order.
- **Completed orders reopening on every reply:** switch to lock-until-revision-requested.
- **No deadlines on time-sensitive services:** set a delivery deadline so due dates and the queue work.
- **Repetitive client questions to the same coworkers:** set up client teams and per-record collaborators so the right people are looped in automatically.
- **Team misses urgent workspace updates:** enable Slack notifications for order, ticket, invoice, and message events if the plan includes Slack.
- **Staff or clients work mostly from phones:** promote the installable app (PWA) and push notifications so updates reach them without email.
- **A board managed by hand:** add pipeline automations so stage changes trigger the next action.
- **"Where's my report?" inquiries:** embed external reports and build client dashboards so results are self-serve.

### Support (clean queue, faster replies)
- **Support coming in by email, untracked:** route it through email-to-ticket so nothing is lost.
- **Repetitive replies typed by hand:** set up saved replies with variables, and a message signature.
- **Replies expected outside business hours:** set an out-of-office auto-reply.
- **Stale tickets cluttering the queue:** use snooze and auto-close for inactive tickets.
- **No quality signal:** enable client ratings to catch unhappy clients early.

### Billing & retention
- **Cancellations handled by support:** turn on self-service cancel/upgrade, and add a cancellation-feedback form to learn why clients leave.
- **Unlimited request services with no cap:** add an active-at-a-time limit and use higher tiers as the upsell.
- **Failed payments not recovering:** confirm dunning and the processor's retry settings are configured.
- **Lumpy cash flow:** offer prepaid account balance or credits.
- **International clients billed in one currency:** set per-service currency, and configure tax for regions you're liable in.

### Scale (low-effort growth)
- **Growth left on the table:** turn on referrals (with commission rules), and consider white-label reselling for agencies or channels that can sell on the agency's behalf.
- **Manual follow-ups slipping:** schedule CRM activities instead of relying on memory, and use the sales pipeline to track deals.

### Operate (trust, safety, automation)
- **Generic workspace URL and branding:** set a custom domain, logo, and colors so the portal feels like the agency's own.
- **Clients land in a generic portal with no guidance:** customize the portal menu, portal dashboard/onboarding builder, and per-client sidebar links before reaching for Twig templates.
- **Everyone sharing one admin login:** create scoped staff roles, including a dedicated, limited account for any AI agent (see `mcp.md`).
- **Repetitive cross-app glue:** connect automations, webhooks, or Zapier/Make so events flow to the agency's other tools.
