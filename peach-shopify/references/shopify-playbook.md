# Peach for Shopify — Playbook

End-to-end recipes for running WhatsApp customer engagement for a Shopify store on
Peach. This document holds the Shopify-specific knowledge; for the underlying tools
and endpoints, defer to the `peach-mcp` and `peach-api` skills.

## Mental model

```
Shopify event (webhook)  →  Peach trigger  →  Stream automation  →  WhatsApp message
                              ├─ condition (the "when", by name)
                              └─ data mappings (pull payload fields via JSONPath)
```

- The **condition** decides when a trigger fires (e.g. "Order Paid"), and may take
  variables (e.g. a field filter so it fires for paid orders that are *not* POS).
- **Data mappings** pull fields out of the Shopify webhook payload into the flow.
  This is the step that most often goes wrong — see the deep dive below.
- The **event source** (your Shopify connection) is derived automatically from the
  condition's integration. It must already be connected in the Peach web app.

## Prerequisites (one-time, in the Peach web app)

1. **Connect Shopify** — create a Shopify custom/private app and add its token under the
   `peach_shopify` integration. This creates the event source triggers attach to.
   Without it, trigger creation returns `409 event_source_not_connected` — that is an
   onboarding action, not something the API/MCP can do.
2. A **live WhatsApp number** (WABA + phone number).
3. At least one **approved template** for business-initiated messages (anything sent
   outside the customer's 24-hour reply window must be a template).

---

# Data mapping deep dive

This is the part that stumps people. Read it before building any trigger.

## The two scopes

A trigger's `data_mappings` is a list. Each entry has a `mapping_for` and a `value`
that maps **output field name → JSONPath into the event payload**:

```json
"data_mappings": [
  { "mapping_for": "recipient", "value": { "phone_number": "$.customer.phone", "name": "$.customer.first_name", "country_code": "$.shipping_address.country_code" } },
  { "mapping_for": "variable",  "value": { "order_id": "$.name", "total": "$.total_price" } }
]
```

- **`recipient`** — *who gets messaged.* Recognized keys: `phone_number` (required),
  `name`, `country_code`, `language`. If `phone_number` resolves to null/blank, the
  trigger fires but no message is sent — this is the #1 silent failure.
- **`variable`** — *data for the message.* Each key becomes a `{{key}}` you can use in
  message text or template liquid values. So `"order_id": "$.name"` lets a step say
  `Your order {{order_id}} is confirmed.`

The variable **key** must match the `{{placeholder}}` in your message/template exactly.

## JSONPath, briefly

- `$` is the root of the webhook payload.
- Dot into objects: `$.customer.first_name`.
- Index into arrays: `$.line_items[0].title`, `$.fulfillments[0].tracking_number`.
- A path that doesn't exist resolves to null — which is exactly how "nothing happened"
  bugs occur.

## The phone-number trap (read this)

Shopify does **not** put a reliable phone number at the top level. `$.phone` is often
`null`. The number can live in several places depending on how the order was created:

| Path | When it's usually present |
| --- | --- |
| `$.phone` | Often null. Don't rely on it alone. |
| `$.customer.phone` | When the customer has a saved phone on their profile. |
| `$.shipping_address.phone` | Most reliable for physical orders. |
| `$.billing_address.phone` | Fallback when shipping has none. |

Guidance:
- For physical-goods stores, prefer `$.shipping_address.phone`, then `$.customer.phone`.
- Always pair it with a country code when the number may be local:
  `"country_code": "$.shipping_address.country_code"` (ISO-2, e.g. `IN`) or
  `$.customer.default_address.country_code`. Peach normalizes the number using the
  account default country code when one isn't supplied, but supplying it is safer.
- **Verify against a real payload** before trusting any single path.

## How to find the right path

Don't guess from memory — capture a real example:
1. In Shopify admin: **Settings → Notifications → Webhooks**, or use a recent order's
   JSON, or send a test webhook.
2. Find the field you want and read off its path.
3. For optional/array fields (tracking numbers, line items), confirm the index exists
   for the order types you care about.

If the user can paste a sample webhook payload, map directly against it rather than
against this cheat-sheet.

## Shopify field cheat-sheet (Order webhooks: orders/paid, orders/create, orders/fulfilled, orders/cancelled)

| You want | JSONPath | Notes |
| --- | --- | --- |
| Order number (display) | `$.name` | e.g. `#1001`. `$.order_number` is the bare integer. |
| Order id (internal) | `$.id` | numeric |
| Total | `$.total_price` | string; pair with `$.currency` |
| Currency | `$.currency` | e.g. `INR` |
| Financial status | `$.financial_status` | `paid`, `pending` (COD) |
| Source | `$.source_name` | `web`, `pos`, `shopify_draft_order` — use to exclude POS |
| Payment gateway | `$.payment_gateway_names[0]` | e.g. `Cash on Delivery (COD)` |
| Customer first name | `$.customer.first_name` | |
| Customer phone | `$.customer.phone` | may be null |
| Shipping phone | `$.shipping_address.phone` | most reliable |
| Shipping country code | `$.shipping_address.country_code` | ISO-2 |
| First line item | `$.line_items[0].title` | array |
| Tracking number | `$.fulfillments[0].tracking_number` | only on fulfilled orders |
| Tracking URL | `$.fulfillments[0].tracking_url` | |
| Cancel reason | `$.cancel_reason` | only on cancelled orders |
| Order status URL | `$.order_status_url` | customer-facing link |
| Abandoned checkout URL | `$.abandoned_checkout_url` | checkout/cart events |

## Common data-mapping mistakes

- **Message shows literal `{{order_id}}`** → no `variable` mapping for `order_id` (or
  the key doesn't match the placeholder). Add/rename the mapping.
- **Trigger fires but no WhatsApp is sent** → `recipient.phone_number` resolved to null.
  Re-map to `$.shipping_address.phone` / `$.customer.phone`.
- **Wrong/garbled number** → missing country code on a local number. Add `country_code`.
- **Empty value for some orders** → the path only exists for certain order types (e.g.
  `$.fulfillments[0].*` is absent until fulfilled). Use the matching event.
- **Currency/format looks off** → `$.total_price` is a raw string ("1999.00"); format it
  in the message copy, not the mapping.

---

# Play 1 — Transactional messages

Order lifecycle notifications. Each is: pick the condition by name → map recipient +
variables → message via a template (business-initiated).

Discover the exact names first:

```text
peach_list_expressions(source: "peach_shopify")
```

Typical conditions (confirm against the account's library):

| Message | Condition name | Variables |
| --- | --- | --- |
| Order confirmation | `Order Paid` | none |
| Order created (incl. COD/pending) | `Order created` | none |
| COD-only, exclude POS | `Order Paid where a field does NOT equal a value` | `key_path`, `value` |
| Shipped / fulfilled | `Order Fulfilled` | none |
| Cancelled | `Order Cancelled` | none |

### Example: order confirmation (order paid)

```json
{
  "stream_id": "strm_...",
  "condition": "Order Paid",
  "data_mappings": [
    { "mapping_for": "recipient", "value": { "phone_number": "$.shipping_address.phone", "name": "$.customer.first_name", "country_code": "$.shipping_address.country_code" } },
    { "mapping_for": "variable",  "value": { "order_id": "$.name", "total": "$.total_price", "currency": "$.currency" } }
  ],
  "activate": true
}
```

Automation step copy (or template body): `Thanks {{name}}! Order {{order_id}} for
{{currency}} {{total}} is confirmed. 🎉`

### Example: COD verification, excluding POS orders

Use the field-filter condition to fire only when the order is *not* a POS sale, then
ask the customer to confirm with quick-reply buttons (Yes/No) and branch.

```json
{
  "stream_id": "strm_...",
  "condition": "Order Paid where a field does NOT equal a value",
  "source": "peach_shopify",
  "variables": { "key_path": "$.source_name", "value": "pos" },
  "data_mappings": [
    { "mapping_for": "recipient", "value": { "phone_number": "$.shipping_address.phone", "name": "$.customer.first_name" } },
    { "mapping_for": "variable",  "value": { "order_id": "$.name", "total": "$.total_price" } }
  ],
  "activate": true
}
```

The automation's first step is a quick-reply question ("Confirm your COD order
{{order_id}}?" → Yes / No) that branches to a confirm or cancel step. (Question nodes
in a Stream support free text, quick-reply buttons, and list messages only.)

### Example: shipping update (fulfilled)

```json
{
  "stream_id": "strm_...",
  "condition": "Order Fulfilled",
  "data_mappings": [
    { "mapping_for": "recipient", "value": { "phone_number": "$.shipping_address.phone", "name": "$.customer.first_name" } },
    { "mapping_for": "variable",  "value": { "order_id": "$.name", "tracking": "$.fulfillments[0].tracking_number", "tracking_url": "$.fulfillments[0].tracking_url" } }
  ],
  "activate": true
}
```

---

# Play 2 — Abandoned-cart recovery with an AI agent

Goal: when a cart/checkout is abandoned, re-engage the customer with a conversational
AI agent (not just a static reminder), so it can answer questions and nudge to checkout.

1. **Identify the condition.** Run `peach_list_expressions` and look for the abandoned-cart
   condition (commonly `Abandoned Cart Created`, under the `Peach` source — abandoned-cart
   detection is a Peach event, not a raw Shopify order webhook). Confirm what's available
   in the account; the exact name/source can differ by store setup.
2. **Build or pick a re-engagement micro-agent** (`peach_create_ai_agent` /
   `peach_list_ai_agents`) — a re-engagement/sales agent with instructions to recover the
   cart, answer objections, and share the checkout link.
3. **Recover into the agent.** Two common shapes:
   - A trigger → automation whose step is a `handoff_ai` to the re-engagement agent, OR
   - `peach_connect_to_ai_agent` with an opening template that re-opens the 24h window,
     then the agent takes over.
4. **Data mappings** for the recipient + the cart context the agent needs:

```json
"data_mappings": [
  { "mapping_for": "recipient", "value": { "phone_number": "$.customer.phone", "name": "$.customer.first_name" } },
  { "mapping_for": "variable",  "value": { "checkout_url": "$.abandoned_checkout_url", "cart_total": "$.total_price", "first_item": "$.line_items[0].title" } }
]
```

The opening template needs an approved MARKETING/UTILITY template (the customer is
outside the 24h window). Pass `checkout_url`, `first_item`, etc. as liquid values so the
first message is personalized, then the agent continues conversationally.

Notes:
- Respect opt-out / marketing communication preferences for abandoned-cart nudges.
- A timed wait before the nudge (e.g. 1 hour) usually performs better — model it as a
  `wait` step in the automation before the handoff.

---

# Play 3 — Support chatbot

Goal: customers message the store on WhatsApp and an AI support agent answers, with
escalation to a human in the shared inbox.

1. **Create a support micro-agent** (`peach_create_ai_agent`) with the store's policies,
   FAQs, return/refund rules, and order-status guidance in its instructions/knowledge.
2. **Entry points:**
   - Inbound, customer-initiated chats: route incoming conversations to the support agent
     (this is typically configured on the phone number's default agent in the web app).
   - Post-purchase support offer: a transactional flow can end with "Need help? Reply
     here" so replies land with the agent inside the 24h window.
3. **Escalation/handoff:** give the agent a handoff path to a human team in the shared
   inbox for anything it can't resolve. Use `peach_list_conversations` /
   `peach_update_conversation` to inspect and route.
4. **Order-status answers:** the agent can use order context already mapped into the
   session, or your `POST /api/v1/orders` / order lookup integration, to answer "where is
   my order".

Support is mostly micro-agent + shared-inbox work; see the `peach-product` and
`peach-mcp` (AI agent tools) skills for agent lifecycle (build → test → publish →
monitor transcripts → refine).

---

# Testing & activation

- Triggers are created as **draft**. Set `activate: true` (on create or via
  `peach_update_trigger`) to go live — activation registers the Shopify webhook for the
  condition's topic.
- **Test** with a real low-value order, or the automation's **Test** share link, before
  relying on live orders.
- **Verify** on the automation's Responses/sessions: confirm the message rendered with
  real values (no literal `{{...}}`), the right recipient, and the expected branch.

# Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `409 event_source_not_connected` | Shopify not connected in the web app | Connect the `peach_shopify` integration first |
| `409 condition_ambiguous` | The condition name exists for >1 integration | Pass `source: "peach_shopify"` |
| `404 condition_not_found` | Wrong/edited condition name | Re-run `peach_list_expressions` and copy the exact `description` |
| `422 missing_variables` | Condition needs variables you didn't supply | Read the expression's `variables` and provide them |
| Trigger active, no message | `recipient.phone_number` is null for this order | Map `$.shipping_address.phone` (then `$.customer.phone`) |
| Literal `{{order_id}}` sent | No `variable` mapping, or key ≠ placeholder | Add/rename the variable mapping |
| Fires for orders it shouldn't | Condition too broad | Use a field-filter condition (e.g. exclude POS via `$.source_name`) |
| Nothing fires at all | Order didn't match the condition, or webhook not registered | Re-check the condition/topic; confirm the trigger is active |
