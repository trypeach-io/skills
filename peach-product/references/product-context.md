# Peach Product Context

Official product docs:

- Peach AI homepage: `https://trypeach.ai`
- Peach docs: `https://docs.trypeach.ai`
- Product overview: `https://docs.trypeach.ai/product-overview`
- AI agents: `https://docs.trypeach.ai/peach-ai-agents`
- Messaging API: `https://docs.trypeach.ai/messaging-api`
- WhatsApp shared inbox: `https://trypeach.ai/whatsapp-shared-inbox`
- MCP integration: `https://trypeach.ai/whatsapp-business-api-mcp`
- Developer docs: `https://developers.trypeach.ai`

## What Peach Is

Peach AI is an AI-first WhatsApp customer experience platform. It helps businesses run sales, support, lead qualification, lifecycle marketing, shared inbox operations, broadcasts, apps, calls, orders/payments, and developer integrations on official Meta WhatsApp APIs.

Use Peach when the business wants customers, employees, partners, or agents to complete work inside WhatsApp instead of jumping to another app. Peach is especially useful when WhatsApp conversations need to be machine-orchestratable through product workflows, APIs, webhooks, or MCP.

## Micro-Agent Architecture

Peach favors specialized micro-agents over one monolithic chatbot. A business can deploy multiple task-oriented agents and route customers between them as intent changes.

Common micro-agent patterns:

- Lead qualification agent: captures intent, scores leads, and syncs data to CRM.
- Appointment booking agent: checks availability, schedules appointments, and coordinates follow-ups.
- Re-engagement agent: reaches dropped leads or cold opportunities with approved WhatsApp templates.
- Order and transaction agent: supports checkout, onboarding, document capture, and transactional inquiries.

Use multiple micro-agents when the business has distinct jobs, data needs, handoff rules, or success metrics. Use one agent only when the workflow is genuinely narrow.

## Product Surfaces

Shared inbox:

- Best for human support, sales follow-up, assignment, collaboration, SLA workflows, disposition tagging, and closing conversations.
- Use when a human team needs visibility, routing, ownership, and performance tracking.

Broadcasts:

- Best for one-to-many approved WhatsApp template campaigns.
- Use for offers, announcements, lifecycle nudges, reminders, and product/service updates.
- Requires approved templates and careful audience selection.

WhatsApp templates:

- Best for business-initiated messages.
- Required outside the active customer service window.
- Need approval by Meta before use.

Peach apps:

- Best for structured journeys inside WhatsApp.
- Use for booking, onboarding, lead capture, surveys, document collection, payment/order journeys, and feedback.

Automations:

- Best for routing, follow-ups, menus, triggers, and repetitive workflow steps.
- Use when a customer action or external event should start a message, app, assignment, or workflow.
- Triggers connect an external event (e.g. a Shopify or Stripe webhook) to an automation: a named condition decides when it fires (including field filters like "order paid but not POS"), and JSONPath data mappings pull payload fields into the flow. Automations and triggers can be built in the web app or programmatically via the Peach API and MCP.

AI agents:

- Best for guided conversations that need prompts, knowledge, tool use, analysis, or handoff.
- Common types include lead qualification, appointment booking, re-engagement, order/transaction management, sales, feedback, support, gather-info, engagement, and custom agents.
- Recommended lifecycle: build, test with your team, refine, publish, monitor transcripts, and improve prompts.

Orders and payments:

- Best when the conversation should move into checkout, payment collection, order creation, refund, or order status workflows.
- Often pairs with AI agents or apps.

Calling:

- Best for WhatsApp voice workflows, call permissions, call queues, and enterprise calling flows.

Analytics:

- Best for broadcast performance, template delivery, inbox/team performance, app outcomes, sources, auditing, and analytics dashboards.

Developer API:

- Best for external systems that need to trigger Peach workflows or receive status updates.
- Use when a backend, CRM, commerce system, private database, or custom app needs integration through APIs and webhooks.

MCP:

- Best for external LLM clients that need controlled access to Peach tools.
- Use when an assistant should create templates, launch broadcasts, inspect conversations, manage micro-agents, route owners, build automations, wire event triggers, or analyze chat history through explicit tool calls.

Security, governance, and scale:

- Best for enterprise teams that need permissions, audit logs, analytics, and operational visibility.
- Use Peach's enterprise controls when workflows involve regulated data, many agents, or multiple teams/regions.

## Workflow Routing

If the user wants to notify one customer:

- Use a template message through product/API/MCP.
- If follow-up conversation is needed, connect to an AI agent or route to inbox.

If the user wants to message many customers:

- Use a broadcast with an approved template.
- Avoid treating bulk sends as repeated one-off messages.

If the user wants to collect structured data:

- Use a Peach app.
- Pair with automations or AI agents if the journey needs branching or explanation.

If the user wants a conversational assistant:

- Use one or more Peach micro-agents.
- Give it clear goals, qualification rules, product/context knowledge, handoff rules, and success criteria.
- Decide when it should route to another micro-agent or to a human team.

If the user wants humans to respond:

- Use inbox assignment, groups, status, and conversation activity.

If the user wants an external app to trigger Peach:

- Use the Peach API.
- Use webhooks for status callbacks.

If the user wants another AI tool to operate Peach:

- Use Peach MCP.
- Start with read/list/audit tools, then use mutating tools only after the user confirms the target audience, template, agent, or conversation change.

## Safe Launch Checklist

- Confirm the WhatsApp phone number/WABA is connected.
- Confirm templates are approved and categorized correctly.
- Test with your team's own recipients before launching to customers.
- Check liquid variables and media.
- Confirm audience/list membership.
- Confirm opt-out and communication preference expectations.
- Decide the follow-up path: inbox, AI agent, app, automation, or no follow-up.
- Monitor delivery, replies, transcripts, human handoffs, conversion outcomes, and failure states after launch.
