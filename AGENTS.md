# AgendaForge for AI agents

## What AgendaForge is

[AgendaForge](https://agendaforge.app) is an AI-native event management platform. Event teams use it to run the full content lifecycle of a conference or corporate event: call for papers and application forms with review pipelines, session and agenda building with tracks, rooms, and conflict checking, speaker management with a task portal, a contact CRM, sponsor management, registration and ticketing, and email communications. Speaker cards and other event assets are generated inside the product.

The platform is agent-friendly by design: every write an agent proposes goes through human approval inside AgendaForge, so an agent can prepare real work (schedule a session, draft an email, update a contact) without ever changing customer data on its own.

## The data model in one paragraph

An **organization** owns **events**. An event has **sessions** (scheduled into **tracks** and **rooms**), **speakers** (who are **contacts** in the organization CRM), **sponsors**, **forms** with submission review **pipelines**, **registrations** with tickets and attendees, a **speaker portal** with tasks, and **email** templates and sends. Most API and MCP calls are scoped to one organization and, from there, one event.

## Integration surfaces

| Surface | Where | Use it for |
| --- | --- | --- |
| REST API v1 | `https://api.agendaforge.app/api/v1` | Programmatic reads and creates, REST Hooks (webhooks) |
| OpenAPI 3.1 spec | https://agendaforge.app/openapi.json | Full endpoint, auth, and error reference |
| Product MCP server | `https://mcp.agendaforge.app/mcp` (read-only variant `/mcp/readonly`) | 51 tools: direct reads plus approval-gated writes |
| Docs MCP server | `https://mcp.agendaforge.app/mcp/docs` | No-auth search and fetch of public docs content |
| Ask endpoint | `https://agendaforge.app/ask?query=...` | Natural-language answers from public content (NLWeb) |
| Markdown mirrors | append `.md` to any page URL | Clean page text for agents; index at https://agendaforge.app/llms.txt and full dump at https://agendaforge.app/llms-full.txt |
| Auth guide | https://agendaforge.app/auth.md | How agents obtain and use credentials |
| Embeds | generated in-product | Public session, speaker, and agenda widgets and JSON feeds |

Zapier, Make, n8n (community node), and a WordPress plugin integrate through the same REST API.

## Authentication

- **Organization API key** (REST and MCP): an Owner or Admin creates it in the dashboard at `dash.agendaforge.app` under **Organization Settings, API tokens**. Keys start with `afk_live_`, are shown once at creation, and carry scopes: `public_api` for the REST API, `mcp:read` and `mcp:write` for MCP. Send as `Authorization: Bearer afk_live_...` or `X-API-Key: afk_live_...`.
- **OAuth 2.1 with PKCE** (MCP): sign in as an AgendaForge user through WorkOS AuthKit. Follow the `401` challenge to the discovery metadata instead of guessing endpoints; the walkthrough is at https://agendaforge.app/auth.md.

## Conventions that matter to agents

- **Approval-gated writes**: MCP write tools create proposals a human approves in AgendaForge. Never report proposed work as done.
- **Errors**: REST errors are one JSON object with a machine-readable `code` and human-readable `message`; every 4xx/5xx is documented in the OpenAPI spec.
- **Retries**: send an `Idempotency-Key` header on REST create calls so a retry can never duplicate a record. On `429`, wait for `Retry-After` seconds.

## Skills in this repository

- [`agendaforge-mcp`](./skills/agendaforge-mcp/SKILL.md): connect an MCP client, authenticate, and use all 51 tools.
- [`agendaforge-api`](./skills/agendaforge-api/SKILL.md): get an API key, call REST API v1, manage REST Hooks, create records safely.
- [`agendaforge-docs`](./skills/agendaforge-docs/SKILL.md): search and fetch public marketing and documentation content, no account required.
- [`agendaforge-webhooks`](./skills/agendaforge-webhooks/SKILL.md): subscribe REST Hook webhooks, verify signed deliveries, handle retries, unsubscribe.
- [`agendaforge-integrations`](./skills/agendaforge-integrations/SKILL.md): connect AgendaForge to Zapier, Make.com, or n8n with an organization API key.
