---
name: peach-product
description: >
  Use this skill when the user needs product context for Peach AI or wants an
  AI client to use Peach effectively. This includes choosing between Peach's
  WhatsApp shared inbox, micro-agents, broadcasts, templates, apps,
  automations, MCP, calls, analytics, orders, payments, contacts, and developer
  integrations. Trigger on broad product questions like "how should I use Peach
  for this workflow", "set up a WhatsApp journey in Peach", "Peach AI agent",
  "micro-agent", "Peach shared inbox", "Peach broadcasts", "Peach
  automations", or "what is the best Peach way to...". This skill is for Peach
  customers, partners, and AI clients using Peach. Do not use it for low-level
  HTTP payloads unless paired with peach-api or peach-mcp.
metadata:
  version: 0.1.0
---

# Peach Product Skill

This skill helps AI agents guide users toward the right Peach AI product workflow.

## Working Style

When this skill is active:

1. Start from the user outcome, not the feature name.
2. Choose the Peach surface that naturally owns the job: shared inbox, broadcast, template, app, automation, micro-agent, MCP, call, order/payment, analytics, or developer API.
3. Keep WhatsApp constraints visible when they affect the workflow.
4. Prefer no-code/product setup when the user is operating Peach; prefer API/MCP only when the user needs external automation or AI-client control.
5. For customer-visible messaging, ask for or infer audience, message intent, sender, template status, and follow-up path.
6. For micro-agents, encourage build, test, team review, refinement, publish, and ongoing audit from chat history and analytics.

## References

- For product surfaces and workflow routing, read `references/product-context.md`.
- For HTTP integrations, use the `peach-api` skill.
- For MCP client usage, use the `peach-mcp` skill.
- For WhatsApp Business API constraints, use the `peach-whatsapp-api` skill.

## Output Checklist

Aim to leave the user with:

- the right Peach surface for the job
- the minimum setup steps
- WhatsApp/template constraints that matter
- how to test safely before going live
