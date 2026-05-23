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
```

Each skill is self-contained. The required file is `SKILL.md`; files under `references/` provide deeper context that an AI agent can load only when needed.

## Using These Skills

Copy or install the desired skill folder into your AI agent or Codex skills directory.

For example, to use only the Peach MCP skill:

```sh
cp -R skills/peach-mcp ~/.codex/skills/
```

To use all skills:

```sh
cp -R skills/peach-* ~/.codex/skills/
```

Exact installation paths may vary depending on your agent runtime.

## Skill Design

These skills follow a concise, progressive-disclosure structure:

1. `SKILL.md` contains trigger guidance, working style, and routing.
2. `references/` contains deeper product, API, MCP, or WhatsApp-specific details.
3. Skills are split by user intent so agents load only the context needed for the task.

## Official References

- Peach AI: https://trypeach.ai
- Peach Docs: https://docs.trypeach.ai
- Peach Developer Docs: https://developers.trypeach.ai
- Peach MCP: https://trypeach.ai/whatsapp-business-api-mcp
- Meta WhatsApp Business Platform Docs: https://developers.facebook.com/docs/whatsapp

