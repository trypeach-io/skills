# Peach HTTP API Reference

Use this reference for external application integrations. Official docs are the source of truth:

- Peach AI homepage: `https://trypeach.ai`
- Developer docs: `https://developers.trypeach.ai`
- Developer docs LLM index: `https://developers.trypeach.ai/llms.txt`
- Product docs: `https://docs.trypeach.ai`
- MCP overview: `https://trypeach.ai/whatsapp-business-api-mcp`

## Base URL And Auth

Production base URL:

```text
https://app.trypeach.io
```

Most public API examples use:

```http
Authorization: <api-key-or-token>
Content-Type: application/json
```

Some account-scoped integrations also support:

```http
X-Account-Api-Key: <account-api-key>
```

For event-driven messaging APIs, prefer the token/header documented for the endpoint because Peach may need to identify the integration/event source, not only the account.

## Main API Families

Messaging:

- `POST /api/v1/events` with `event_type: "send_template_message"` sends an approved WhatsApp template message.
- `GET /api/v1/events/:id` polls event status.
- `GET /api/v1/template_messages` polls delivery status for sent template messages.
- `POST /api/v1/events` with `event_type: "send_broadcast"` launches a broadcast.
- `POST /api/v1/events` with `event_type: "connect_to_ai_agent"` sends a template and connects the contact to a Peach micro-agent.
- `POST /api/v1/events` with `event_type: "send_app_message"` sends a Peach app/flow message.

WhatsApp templates:

- `GET /api/v1/whats_app_templates`
- `POST /api/v1/whats_app_templates`
- `GET/PATCH/DELETE /api/v1/whats_app_templates/:id`
- `PATCH /api/v1/whats_app_templates/:id/archive`
- `PATCH /api/v1/whats_app_templates/:id/pause`
- `PATCH /api/v1/whats_app_templates/:id/submit`

Contacts:

- `POST /api/v1/contacts`
- `PATCH /api/v1/contacts/:phone_number`
- `POST /api/v1/contacts/bulk_update_communication_preferences`

Orders and refunds:

- `POST /api/v1/orders`
- `GET /api/v1/orders/:id`
- `POST /api/v1/orders/:order_id/refunds`

Media:

- `GET /api/v1/medias`
- `POST /api/v1/medias`
- `DELETE /api/v1/medias/:id`
- `POST /api/v1/medias/direct_uploads`
- `POST /api/v1/medias/finalize`

Conversations and activities:

- `GET /api/v1/conversations`
- `PATCH /api/v1/conversations/:id`
- `GET /api/v1/activities`

Setup links:

- `POST /api/v1/setup_guest_links`
- `DELETE /api/v1/setup_guest_links/:id`
- `POST /api/v1/setup_guest_links/:id/regenerate`

Automations (Streams) — step-graph conversation flows:

- `GET /api/v1/streams` (optionally filter by `execution_mode`)
- `POST /api/v1/streams` — create (identify the sending number with `business_phone_number`, the wa_id; supply a step-graph `definition`)
- `GET /api/v1/streams/:id`
- `PATCH /api/v1/streams/:id` — update name/description/status/definition (set `status: "published"` to publish)
- `POST /api/v1/streams/:stream_id/events` — fire an event into an automation for a contact

Triggers and conditions — start an automation from an external event:

- `GET /api/v1/expressions` — the condition library (optionally filter by `source`, e.g. `peach_shopify`); each returns a friendly `description`, `topic`, and the `variables` it needs
- `GET /api/v1/streams/:stream_id/triggers`
- `POST /api/v1/streams/:stream_id/triggers` — create a trigger (by condition name + data mappings)
- `PATCH /api/v1/triggers/:id` — update variables / data mappings / activation

## Send A Template Message

Use an approved WhatsApp template. The template can include named liquid values supplied at send time.

```json
{
  "event_type": "send_template_message",
  "contact": {
    "name": "Alfred Hitchcock",
    "email": "alfred@example.com",
    "phone_number": "+919988776655",
    "metadata": {
      "customer_tier": "gold"
    }
  },
  "template_message": {
    "whats_app_template_id": "wat_...",
    "liquid_values": {
      "product_name": "Product 1",
      "price": 112,
      "currency": "INR",
      "header_media": "https://example.com/image.png"
    },
    "business_phone_number": "919876543222",
    "reply_automation": {
      "app_id": "aiflw_..."
    }
  }
}
```

Useful checks before sending:

- The template is approved by Meta.
- Every required placeholder has a non-blank liquid value.
- Header media values match the template media type.
- The selected business phone number belongs to the account/WABA.
- The contact has not opted out of the relevant message type.

## Launch A Broadcast

Broadcasts are for sending an approved template to many contacts. Choose either explicit contacts or an existing audience/list, depending on the endpoint payload.

Useful checks:

- Use an approved template.
- Validate each phone number, or intentionally enable invalid-number skipping when the API supports it.
- Provide per-contact liquid values when personalization differs by recipient.
- Prefer scheduled or smart delivery modes for large sends when supported.
- Poll broadcast/event status after launch.

## Connect To A Micro-Agent

Use `connect_to_ai_agent` when the desired workflow is: send an approved template, then route the contact into a Peach micro-agent.

Required concepts:

- Contact: usually `phone_number` plus optional name/email/metadata.
- Template message: approved template plus liquid values.
- Micro-agent: usually an `aiflw_...` ID.

Do not use this for generic chatbot strategy; use the Peach product skill for micro-agent setup, routing, and prompt guidance.

## Trigger An Automation From An Event

Run an automation when an external event arrives (e.g. a Shopify order is paid). Two parts: a **condition** (the "when") and **data mappings** (pull fields out of the event payload with JSONPath into the flow).

1. Connect the source integration (e.g. Shopify) in the Peach web app first. The event source is derived from the condition's integration; the API never takes an event-source ID. If the integration is not connected, trigger creation returns `409` with `code: "event_source_not_connected"` — an onboarding step, not an API fix.
2. List the condition library to find the condition by name and its variables:

```text
GET /api/v1/expressions?source=peach_shopify
```

3. Create the trigger by condition **name** (not an internal ID). Use `source` only to disambiguate when the same name exists for multiple integrations:

```json
{
  "condition": "Order Paid where a field does NOT equal a value",
  "source": "peach_shopify",
  "variables": { "key_path": "$.source_name", "value": "pos" },
  "data_mappings": [
    { "mapping_for": "recipient", "value": { "phone_number": "$.customer.phone", "name": "$.customer.first_name" } },
    { "mapping_for": "variable",  "value": { "order_id": "$.id", "total": "$.total_price" } }
  ],
  "activate": true
}
```

`POST /api/v1/streams/:stream_id/triggers`. `recipient` mappings decide who is messaged; `variable` mappings become `{{key}}` values inside the flow's steps. Triggers are created as drafts unless `activate: true`; activation registers the integration webhook.

Notes:

- Identify conditions by `condition` (the expression's `description`). `expression_id` (`exp_...`) is accepted as an advanced fallback.
- Required `variables` come from the chosen expression; missing ones return `422` with `code: "missing_variables"`.
- An unknown name returns `404` `condition_not_found`; a shared name returns `409` `condition_ambiguous` with candidate sources — pass `source`.

## Webhooks

Use webhooks when the external app needs delivery, order, flow execution, or micro-agent execution updates.

Common webhook families in the developer docs:

- order status webhooks
- flow execution status webhooks
- message delivery status webhooks
- AI agent/micro-agent execution webhooks

When implementing a receiver:

- Verify Peach's documented signature or secret mechanism when available.
- Make handlers idempotent.
- Return quickly and process expensive work asynchronously.
- Store event IDs or delivery IDs so retries do not duplicate side effects.

## Error Handling

Common failures:

- `401 Unauthorized`: missing, invalid, expired, or wrong kind of API credential.
- `404 Not found`: account-scoped object not found, wrong prefix ID, or unavailable event source.
- `422 Parameter Error`: invalid payload, missing required field, invalid template variables, invalid phone number, or an object in the wrong state.

For messaging failures, distinguish:

- API request accepted by Peach
- Peach event processed
- WhatsApp template message queued
- Meta delivery status delivered/read/failed

Do not treat request acceptance as final delivery.
