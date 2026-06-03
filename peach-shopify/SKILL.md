---
name: peach-shopify
description: >
  Use this skill when a Shopify merchant wants to run WhatsApp customer
  engagement through Peach: transactional messages (order confirmation, COD
  verification, shipping/out-for-delivery, cancellation), abandoned-cart
  recovery with an AI agent, and a support chatbot. It is the end-to-end
  playbook that composes Peach automations, triggers/conditions, data mappings,
  templates, and micro-agents for the Shopify use case. Trigger on phrases like
  "Shopify WhatsApp", "WhatsApp order confirmation", "abandoned cart recovery on
  WhatsApp", "COD verification WhatsApp", "Shopify support chatbot", "Peach +
  Shopify", or "set up WhatsApp for my store". Especially relevant when the user
  is stuck on mapping Shopify webhook fields (phone number, order id, totals)
  into a Peach flow.
metadata:
  version: 0.1.0
---

# Peach for Shopify Skill

This skill helps set up WhatsApp customer engagement for a Shopify store on Peach,
end to end. It is a use-case playbook — it composes the underlying Peach
capabilities rather than re-documenting them.

## How this skill relates to the others

- For exact MCP tools and call sequences, use the `peach-mcp` skill (`references/tools.md`).
- For exact HTTP endpoints and payloads, use the `peach-api` skill (`references/http-api.md`).
- For WhatsApp template/window/category rules, use the `peach-whatsapp-api` skill.
- For choosing between inbox, broadcast, app, automation, or micro-agent, use the `peach-product` skill.

Use whichever transport the user has connected: Peach MCP tools (e.g. `peach_create_trigger`)
or the HTTP API (e.g. `POST /api/v1/streams/:id/triggers`). The concepts and
payloads are identical.

## Working Style

When this skill is active:

1. Confirm the one-time setup is done first: Shopify is connected in the Peach web app
   (private app token on the `peach_shopify` integration), a WhatsApp number is live,
   and at least one approved template exists for business-initiated sends.
2. **Discover, don't guess.** Always run `peach_list_expressions` (filter by
   `source: "peach_shopify"`) for the real condition names and their required
   variables in *this* account before creating a trigger. Condition libraries vary.
3. Reference conditions by their friendly **name** (`condition`), not internal IDs.
   Identify the sending number with `business_phone_number` (wa_id).
4. **Get the data mapping right** — this is where setups fail. Always inspect a real
   Shopify webhook payload, confirm where the phone number actually lives, and map
   `recipient` (who to message) and `variable` (`{{key}}` values) explicitly. See
   `references/shopify-playbook.md` → "Data mapping deep dive".
5. Build triggers as draft, review, then activate (activation registers the Shopify webhook).
6. Treat every outbound flow as customer-visible: confirm template approval, the 24h
   window, and opt-out assumptions before going live.

## References

- For the full playbook — prerequisites, the three plays (transactional, cart
  recovery, support), the Shopify field/JSONPath cheat-sheet, and data-mapping
  troubleshooting — read `references/shopify-playbook.md`.

## Output Checklist

Aim to leave the user with:

- which condition (by name) + variables to use for each Shopify event
- the exact `recipient` and `variable` data mappings, with correct JSONPaths
- the template or message copy, with `{{placeholders}}` that match the variable keys
- how to test (a real test order or the Test share link) and where to verify (Responses)
- the activation step and what it does (registers the Shopify webhook)
