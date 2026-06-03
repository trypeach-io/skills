---
name: peach-api
description: >
  Use this skill when the user wants to integrate with Peach from external
  application code, backend services, workflow builders, webhook handlers, or
  server-side automation. This includes Peach HTTP API workflows for WhatsApp
  template messages, broadcasts, contacts, templates, micro-agents, app messages,
  orders, media, conversations, setup links, status polling, webhooks, automations
  (Streams), and event-driven triggers that start an automation from a Shopify or
  Stripe webhook using a named condition and JSONPath data mappings. Trigger on
  phrases like "Peach API", "developers.trypeach.ai", "send a WhatsApp template
  through Peach", "launch a Peach broadcast", "create a Peach contact",
  "Peach automation/trigger", "Shopify order to WhatsApp", or "integrate Peach
  into my app".
metadata:
  version: 0.2.0
---

# Peach API Skill

This skill helps external developers use Peach's HTTP APIs correctly from application code.

## Working Style

When this skill is active:

1. Identify the workflow first: messaging, templates, contacts, broadcasts, orders, media, conversations, micro-agents, automations/triggers, or webhooks.
2. Prefer the official developer reference for exact endpoints and payloads.
3. Keep API keys and tokens server-side. Do not suggest exposing Peach credentials in browser code, mobile apps, or client-side workflow builders.
4. Use prefix IDs at API boundaries when the endpoint expects them, such as `wat_`, `cmp_`, or `aiflw_`.
5. Treat phone numbers as account-scoped customer identifiers; use E.164 format for outbound messaging.
6. For WhatsApp sends, check template approval, messaging window, liquid values, phone number, and opt-out/communication preference assumptions.
7. For asynchronous operations, return or poll the Peach event/message/broadcast status rather than assuming immediate delivery.
8. For automations and triggers, reference conditions by their friendly name (`condition`) rather than internal IDs, identify the sending number with `business_phone_number` (wa_id), and remember the source integration must be connected in the Peach web app before a trigger can activate.

## References

- For endpoints, auth, payloads, status polling, webhooks, and examples, read `references/http-api.md`.
- For WhatsApp Business API restrictions and message-format choices, use the `peach-whatsapp-api` skill.
- For MCP-client usage, use the `peach-mcp` skill.

## Output Checklist

Aim to leave the user with:

- the correct Peach API endpoint or workflow
- the minimal required payload shape
- the auth header/token placement
- Peach/WhatsApp caveats that affect whether the request will work
- a small validation step, usually a status poll or test send
