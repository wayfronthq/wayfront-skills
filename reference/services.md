# Wayfront Services

A **service** is an item in the workspace's product catalogue — what clients buy. Purchasing a service produces an **order** (and, for recurring services, a **subscription**). Services drive pricing, the intake form a client fills out, and the delivery workflow.

## Key Fields

Always confirm the live shape via the MCP `ServiceTool` schema or `wayfront api list Services` — fields evolve. The core set:

| Field | Purpose |
|---|---|
| `name`, `description`, `image` | Display in the catalogue |
| `price`, `pretty_price`, `currency` | One-off price + display string |
| `recurring` | Billing mode: `0` one-off · `1` recurring (no trial/setup) · `2` recurring with trial or setup fee |
| `f_price`, `f_period_l`, `f_period_t` | **First** period: price, length, type (used for trial / setup-fee when `recurring = 2`) |
| `r_price`, `r_period_l`, `r_period_t` | **Recurring** period: price, length, type (day/week/month/year) |
| `recurring_action` | What happens each renewal: `0` reopen · `1` create order · `2` reset requests · `3` do nothing |
| `max_active_requests`, `request_orders`, `multi_order` | How many concurrent orders/requests a subscription allows |
| `option_categories`, `option_variants` | Variant/option matrix (a service with these is a "variant service") |
| `intake_form_id` | Form the client fills out at checkout (see `FormTool`) |
| `is_taxable` | Whether tax applies |
| `public`, `sort_order`, `folder_id` | Visibility, ordering, catalogue folder |
| `provider_id`, `hoth_product_key`, `braintree_plan_id` | Whitelabel / fulfilment / payment integration keys |

### Billing model cheatsheet
- **One-off:** `recurring = 0`, set `price`.
- **Simple subscription:** `recurring = 1`, set `r_price` + `r_period_l`/`r_period_t` (e.g. `1` / `month`).
- **Subscription with trial or setup fee:** `recurring = 2`, set the `f_*` first-period fields *and* the `r_*` recurring fields. For mixing services in one subscription, their `r_period_*` (and `f_period_*` where present) must be compatible — the app rejects incompatible billing periods.

## Set Up a Service

**Via CLI:**
```bash
wayfront services list
wayfront services get 12
wayfront services create --json '{
  "name": "SEO Audit",
  "description": "One-time technical SEO audit",
  "price": 499,
  "currency": "USD",
  "recurring": 0,
  "is_taxable": true,
  "public": true
}'
wayfront services update 12 --json '{"price": 599}'
```

Recurring example:
```bash
wayfront services create --json '{
  "name": "Managed SEO",
  "recurring": 1,
  "r_price": 1500,
  "r_period_l": 1,
  "r_period_t": "month",
  "recurring_action": 1,
  "currency": "USD",
  "public": true
}'
```

**Via raw API** (future-proof): discover with `wayfront api list Services`, then `wayfront api call servicesStore --json '{...}'`.

**Via MCP:** `ServiceTool` `list`/`show` to inspect; `create`/`update` where your token permits. Read-only support deployments **block** service writes — use the CLI/API with a write-capable session for setup.

## Workflow Notes

- **Attach an intake form** by setting `intake_form_id` to a form from `FormTool`/`wayfront api list` — that's what the client fills out at checkout, surfacing later as the order's filled form fields.
- **Variants:** populate `option_categories` + `option_variants` for tiered/optional pricing; such services are treated as variant services.
- **Visibility:** `public: false` keeps a service orderable internally / by direct link without listing it in the public catalogue.
- **Folders:** group catalogue items with `folder_id`.
- Confirm a new service end-to-end: create it, place a test order in the portal, and check the resulting order/subscription and invoice.
