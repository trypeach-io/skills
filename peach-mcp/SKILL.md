---
name: peach-mcp
description: >
  Use this skill when the user wants an AI client or MCP client to use Peach
  product capabilities through Peach's native MCP server or Peach MCP tools. This
  includes sending WhatsApp template messages, launching broadcasts, managing
  contacts/templates/media, connecting contacts to micro-agents, reading and
  analyzing messages, updating conversations, creating setup links, auditing or
  improving agent prompts, building automations (Streams), and wiring triggers
  that start an automation from an external event (e.g. a Shopify or Stripe
  webhook) with conditions and JSONPath data mappings, and using Peach
  effectively via tool calls. Trigger on phrases like "Peach MCP", "MCP client",
  "use Peach tools", "peach_send_template_message", "peach_create_trigger",
  "Shopify WhatsApp automation", "Claude/ChatGPT MCP with Peach", or "let an
  AI agent operate Peach".
metadata:
  version: 0.2.0
---

# Peach MCP Skill

This skill helps AI/MCP clients operate Peach AI safely and productively.

## Working Style

When this skill is active:

1. Translate the user's goal into the smallest safe Peach tool call sequence.
2. Prefer read/list tools before mutating tools when object IDs, template variables, or account state are uncertain.
3. For outbound WhatsApp actions, confirm template approval, recipient phone number, liquid values, sender phone number, and intent.
4. Treat tool results as operational state, not conversational truth. Poll or list status when the workflow is asynchronous.
5. Avoid leaking API keys, OAuth tokens, raw credentials, customer PII, or private transcript data into prompts or logs.
6. Use product language the user understands: template, broadcast, contact, micro-agent, app message, conversation, setup link.

## References

- For available tools, inputs, routing, and common sequences, read `references/tools.md`.
- For WhatsApp Business API restrictions, use the `peach-whatsapp-api` skill.
- For external HTTP API integrations instead of MCP clients, use the `peach-api` skill.

## Output Checklist

Aim to leave the user with:

- the tool or tool sequence to use
- required IDs and fields to gather first
- safety checks before irreversible or customer-visible actions
- how to verify the result after the tool call
