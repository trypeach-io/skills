# Peach Skills

AI skills for using Peach AI effectively from external assistants, MCP clients, and developer integrations.

Peach AI is an AI-first WhatsApp customer experience platform for shared inbox workflows, micro-agents, broadcasts, templates, automations, analytics, developer APIs, and native MCP-based orchestration.

## Skills

This repository contains separate skills for different Peach use cases:

| Skill | Purpose |
| --- | --- |
| `peach-product` | Product context for choosing the right Peach workflow, such as shared inbox, broadcasts, templates, apps, micro-agents, MCP, analytics, and developer integrations. |
| `peach-api` | External developer guidance for integrating with Peach HTTP APIs from backend services, webhook handlers, workflow builders, and server-side automation. |
| `peach-mcp` | Guidance for MCP clients and external LLMs operating Peach through Peach's native MCP tools. |
| `peach-whatsapp-api` | WhatsApp Business Platform constraints as they apply to Peach, including templates, categories, WABA/phone setup, messaging windows, delivery status, and broadcasts. |
| `peach-shopify` | Use-case playbook for setting up WhatsApp engagement for a Shopify store: transactional messages, abandoned-cart recovery with an AI agent, and a support chatbot — including the Shopify webhook data-mapping details. |

## Repository Structure

```text
skills/
  peach-api/
    SKILL.md
    references/
      http-api.md
  peach-mcp/
    SKILL.md
    references/
      tools.md
  peach-product/
    SKILL.md
    references/
      product-context.md
  peach-whatsapp-api/
    SKILL.md
    references/
      whatsapp-platform.md
  peach-shopify/
    SKILL.md
    references/
      shopify-playbook.md
```
