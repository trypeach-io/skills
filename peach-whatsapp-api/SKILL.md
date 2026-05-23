---
name: peach-whatsapp-api
description: >
  Use this skill when the user needs help understanding or applying WhatsApp
  Business Platform rules through Peach. This includes template approval,
  template categories, 24-hour messaging windows, free-form versus template
  messages, WhatsApp Flows, phone number/WABA setup, sender selection, opt-outs,
  delivery restrictions, broadcasts, and when to use Peach APIs or MCP tools for
  WhatsApp messaging. Trigger on phrases like "WhatsApp API", "WhatsApp
  Business API", "WABA", "template category", "24-hour window", "broadcast on
  WhatsApp", "why did WhatsApp reject/fail this message", or "send WhatsApp
  messages through Peach". This skill is for using official Meta WhatsApp APIs
  effectively through Peach, including AI-first WhatsApp CX workflows.
metadata:
  version: 0.1.0
---

# Peach WhatsApp API Skill

This skill helps AI agents reason about WhatsApp Business Platform constraints as they apply to Peach AI.

## Working Style

When this skill is active:

1. Decide whether the desired message is business-initiated or user-initiated.
2. If business-initiated, use an approved template.
3. If inside the 24-hour customer service window, free-form replies may be possible through the product/API path that supports them.
4. Choose the right template category: `UTILITY`, `MARKETING`, or `AUTHENTICATION`.
5. Check WABA/phone number setup before diagnosing delivery issues.
6. Separate Peach request success from WhatsApp delivery success.
7. When exact Meta policy or pricing behavior matters, verify against current Meta/Peach docs.

## References

- For WhatsApp concepts, restrictions, setup, and troubleshooting, read `references/whatsapp-platform.md`.
- For sending through Peach HTTP APIs, use the `peach-api` skill.
- For sending through MCP tools, use the `peach-mcp` skill.

## Output Checklist

Aim to leave the user with:

- whether a template is required
- the right template category/message format
- setup prerequisites such as WABA, phone number, template approval, and sender number
- likely failure causes and a verification step
