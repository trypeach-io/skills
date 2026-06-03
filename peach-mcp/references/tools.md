# Peach MCP Tool Reference

Peach AI's MCP server lets external LLM clients operate WhatsApp customer experience workflows through explicit tools. Use MCP when the assistant should inspect, manage, or act inside Peach rather than merely describe what a human should click.

Peach exposes MCP over JSON-RPC at:

```text
POST https://app.trypeach.io/api/mcp
```

The server supports:

- `initialize`
- `notifications/initialized`
- `tools/list`
- `tools/call`

Authentication may use an OAuth bearer token, account API key header, or legacy token depending on the client setup. OAuth-capable clients should follow the protected-resource metadata returned by unauthorized MCP responses.

## Tool Families

Messaging and events:

- `peach_send_template_message`
- `peach_launch_broadcast`
- `peach_send_test_template_message`
- `peach_connect_to_ai_agent`
- `peach_send_app_message`
- `peach_get_event_status`
- `peach_list_template_messages`

Templates:

- `peach_list_templates`
- `peach_get_template`
- `peach_create_template`
- `peach_update_template`
- `peach_archive_template`
- `peach_pause_template`
- `peach_submit_template`

Contacts:

- `peach_create_contact`
- `peach_create_contacts`
- `peach_update_contact`

Setup guest links:

- `peach_create_setup_guest_link`
- `peach_expire_setup_guest_link`
- `peach_regenerate_setup_guest_link`

Messages and media:

- `peach_list_messages`
- `peach_list_media`
- `peach_upload_media`
- `peach_delete_media`

Broadcasts:

- `peach_list_broadcasts`
- `peach_get_broadcast`
- `peach_cancel_broadcast`
- `peach_archive_broadcast`

Conversations:

- `peach_list_conversations`
- `peach_list_activities`
- `peach_update_conversation`

AI agents:

- `peach_list_ai_agents`
- `peach_get_ai_agent`
- `peach_create_ai_agent`
- `peach_update_ai_agent`
- `peach_list_ai_agent_sessions`
- `peach_get_ai_agent_transcript`

Use AI-agent tools for Peach micro-agents: lead qualification, appointment booking, re-engagement, order/transaction management, sales, support, feedback, gather-info, engagement, and custom agents.

Automations (Streams) — step-graph conversation flows:

- `peach_list_automations`
- `peach_get_automation`
- `peach_create_automation`
- `peach_update_automation`
- `peach_trigger_automation`

Triggers and conditions — start an automation from an external event (e.g. a Shopify or Stripe webhook):

- `peach_list_expressions`
- `peach_list_triggers`
- `peach_create_trigger`
- `peach_update_trigger`

A **trigger** wires an automation to an event. It has two parts:

- A **condition** (the "when") chosen from the expression library by its friendly name, e.g. `"Order Paid"`. Some conditions take variables (e.g. a field/value filter for "order paid but source is not POS").
- **Data mappings** that pull fields out of the event payload using JSONPath: `recipient` mappings decide who is messaged; `variable` mappings become `{{key}}` values inside the flow.

The event source (the connected integration, e.g. Shopify) is derived automatically from the chosen condition — callers never pass an event-source ID. The integration must already be connected in the Peach web app; if it is not, `peach_create_trigger` returns an "event source not connected" error that is an onboarding step, not an API action.

## Common Workflows

Send a template message:

1. Use `peach_list_templates` or `peach_get_template` if the template ID/name is uncertain.
2. Verify the template is approved and identify required placeholders.
3. Call `peach_send_template_message` with `contact` and `template_message`.
4. Use `peach_get_event_status` or `peach_list_template_messages` to check processing/delivery.

Launch a broadcast:

1. Use `peach_list_templates` to select an approved template.
2. Decide whether recipients come from explicit contacts or an existing audience/list.
3. Call `peach_launch_broadcast`.
4. Use `peach_get_event_status`, `peach_list_broadcasts`, or `peach_get_broadcast` to verify state.

Connect a contact to an AI agent:

1. Use `peach_list_ai_agents` or `peach_get_ai_agent` to identify the agent.
2. Use `peach_list_templates` to choose an approved opening template.
3. Call `peach_connect_to_ai_agent`.
4. Check event/template-message status and later inspect sessions if needed.

Audit or improve a micro-agent:

1. Use `peach_list_ai_agents` to find the agent.
2. Use `peach_get_ai_agent` to inspect instructions and configuration.
3. Use `peach_list_ai_agent_sessions` and `peach_get_ai_agent_transcript` to review real conversations.
4. Propose prompt or routing improvements before calling `peach_update_ai_agent`.

Send a test template:

1. Use `peach_send_test_template_message`.
2. Prefer `generate_share_link: true` when the user wants a user-initiated WhatsApp test link or wants to avoid direct-send constraints.
3. Use direct send only when the recipient and sender context are clear.

Manage templates:

1. Use `peach_create_template` with `template_name`, `language`, `category`, and `components_attributes`.
2. Include at least a body component.
3. Use `peach_submit_template` only when the draft is ready for Meta review.
4. Use `peach_pause_template` or `peach_archive_template` for templates that should stop being used.

Review conversations:

1. Use `peach_list_conversations` to find the conversation.
2. Use `peach_list_messages` and `peach_list_activities` for context.
3. Use `peach_update_conversation` only after deciding the desired assignment/status change.

Operate the shared inbox:

1. Use `peach_list_conversations` to find active work.
2. Use `peach_list_messages` and `peach_list_activities` for context.
3. Route or update conversations only after checking owner, status, and user intent.

Build an event-triggered automation (e.g. WhatsApp message when a Shopify order is paid):

1. Create or identify the automation with `peach_create_automation` (definition-driven step graph) or `peach_list_automations`/`peach_get_automation`.
2. Use `peach_list_expressions` (optionally filter by `source`, e.g. `peach_shopify`) to find the condition by name and see which variables it needs.
3. Call `peach_create_trigger` with the automation `stream_id`, the `condition` name, any `variables`, and `data_mappings` (JSONPath into the event payload). The trigger is created as a draft.
4. Review with `peach_list_triggers`, then set it live by calling `peach_create_trigger` with `activate: true` (or `peach_update_trigger` with `activate: true`). Activation registers the integration webhook.
5. Trigger a test with `peach_trigger_automation`, or place a real test order, and verify via the automation's responses/sessions.

## Safety Rules

- Treat `peach_send_template_message`, `peach_launch_broadcast`, `peach_connect_to_ai_agent`, and `peach_send_app_message` as customer-visible actions.
- For bulk operations, preview recipients and template variables before launching.
- Do not create or submit WhatsApp templates with policy-sensitive content unless the user has reviewed the copy.
- Do not update or close conversations without understanding the current owner/status when an agent team is involved.
- Do not expose transcript or contact details outside the user's stated purpose.
- For agent-prompt updates, summarize the intended behavior change and preserve existing business constraints.

## Useful Input Notes

Contacts usually require:

```json
{
  "phone_number": "+14155552671",
  "name": "Ada Lovelace",
  "email": "ada@example.com",
  "metadata": {
    "plan": "pro"
  }
}
```

Template messages usually require:

```json
{
  "whats_app_template_id": "wat_...",
  "liquid_values": {
    "first_name": "Ada"
  },
  "business_phone_number": "14155552671"
}
```

AI agents usually use IDs like:

```text
aiflw_...
```

Broadcasts usually use IDs like:

```text
cmp_...
```

Automations usually use IDs like:

```text
strm_...
```

Creating an automation identifies the sending number by `business_phone_number` (WhatsApp wa_id format, e.g. `919876543210`), the same as template/broadcast sends — not an internal numeric ID:

```json
{
  "name": "Order paid thank-you",
  "business_phone_number": "919876543210",
  "definition": {
    "start_step": "thanks",
    "steps": {
      "thanks": {
        "type": "send_message",
        "payload": { "message": { "text": "Thanks! Order {{order_id}} is confirmed." } },
        "next_step": null
      }
    }
  }
}
```

Creating a trigger references the condition by its friendly name (not an internal ID). Use `source` only to disambiguate when the same name exists for more than one integration:

```json
{
  "stream_id": "strm_...",
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
