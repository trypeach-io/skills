# WhatsApp Business Platform In Peach

Official references:

- Peach AI homepage: `https://trypeach.ai`
- Peach WhatsApp docs: `https://docs.trypeach.ai/whatsapp-business-api`
- Message formats: `https://docs.trypeach.ai/message-formats`
- Template categorization: `https://docs.trypeach.ai/whatsapp-template-categorization`
- Phone number setup: `https://docs.trypeach.ai/phone-number-requirements`
- MCP overview: `https://trypeach.ai/whatsapp-business-api-mcp`
- Meta WhatsApp docs: `https://developers.facebook.com/docs/whatsapp`

## Core Concepts

WABA:

- WhatsApp Business Account.
- Owns templates, phone numbers, and messaging capability.
- Usually connected to Peach through Meta/Facebook onboarding.

Business phone number:

- The WhatsApp sender identity.
- One account can have multiple business phone numbers.
- APIs and tools may require a specific sender when multiple numbers are available.

Contact:

- The recipient/customer.
- Use E.164 phone numbers for API/MCP sends.
- Do not assume phone numbers are globally unique outside the Peach account.

## Message Types

Template messages:

- Pre-approved by Meta.
- Required for business-initiated outreach and for messages outside the active service window.
- Can include body text, header media, footer, buttons, and variables.
- In Peach, variables are commonly supplied as `liquid_values`.

Free-form messages:

- Used for ongoing customer conversations inside the customer service window.
- Do not require template approval.
- Usually best handled by Peach inbox, automations, AI agents, or product-specific reply flows rather than generic outbound APIs.

WhatsApp Flows / Peach apps:

- Structured, app-like journeys inside WhatsApp.
- Useful for booking, lead capture, payments, feedback, onboarding, and data collection.
- In Peach, app messages can be triggered through product workflows, API, or MCP where supported.

## Template Categories

`UTILITY`:

- Transactional or account-related updates.
- Examples: order confirmation, shipping update, appointment reminder, payment receipt.

`MARKETING`:

- Promotional, re-engagement, offers, announcements, product nudges, or content that encourages a commercial action.
- Meta may recategorize templates if the content is promotional.

`AUTHENTICATION`:

- OTPs and login/account verification flows.
- Keep content constrained to authentication.

When uncertain, choose the category that matches the actual user-visible copy, not the business intent behind it.

## Common Restrictions

- A business usually cannot start a WhatsApp conversation with arbitrary free text; use an approved template.
- Templates must be approved before send.
- Template placeholders must be provided and non-blank.
- Header media must match the approved header type.
- Recipients must be valid WhatsApp users and reachable.
- Users may opt out or block the business.
- Meta may rate-limit, quality-limit, pause, reject, or recategorize templates.
- Delivery status can fail after Peach accepts the request.

## Choosing A Peach Path

Use Peach API/MCP template send when:

- sending transactional notifications
- initiating a customer conversation
- sending approved campaign or lifecycle messaging
- connecting a contact to an AI agent after an approved opening template

Use Peach broadcasts when:

- sending an approved template to many contacts
- needing campaign state, audience handling, analytics, and delivery tracking

Use Peach AI agents when:

- the user wants an ongoing WhatsApp conversation handled by prompts, knowledge, tools, and handoff rules
- the business needs specialized micro-agents for lead qualification, appointment booking, re-engagement, or transactional support

Use Peach apps/flows when:

- structured data capture or a transactional journey is better than free-form chat

Use the inbox when:

- a human team needs to reply, assign, close, or inspect conversations

## Troubleshooting

If a send fails, check in this order:

1. Was the Peach API/MCP request authenticated and accepted?
2. Did Peach create/process an event or template message?
3. Is the template approved and active?
4. Are all liquid/template values present?
5. Is the business phone number correct for the WABA/template?
6. Is the recipient phone number valid and in E.164 format?
7. Has the user opted out, blocked the number, or exceeded policy/frequency constraints?
8. Does Meta return a delivery or quality-related error?

For debugging, always keep the difference clear between Peach event status, Peach template-message status, and Meta/WhatsApp delivery status.
