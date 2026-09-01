# AgendaForge skills

Official skills for [AgendaForge](https://agendaforge.app), the AI-native event management platform: call for papers and review, agenda building, speaker and sponsor management, registration, and event email, with an MCP server and REST API built for AI agents.

Use these skills to help an AI agent connect to the [AgendaForge MCP server](https://agendaforge.app/developers) or integrate with the REST API.

Install them with:

```sh
npx skills add agendaforge/skills
```

## Skills

- [`agendaforge-mcp`](./skills/agendaforge-mcp/SKILL.md): Connect an MCP client, choose authentication, use read tools, and submit approval-gated write proposals.
- [`agendaforge-api`](./skills/agendaforge-api/SKILL.md): Authenticate to REST API v1, test a connection, manage REST Hooks, and create contacts, sessions, and sponsors.
- [`agendaforge-docs`](./skills/agendaforge-docs/SKILL.md): Search and fetch AgendaForge public marketing and documentation content, no account required.
- [`agendaforge-webhooks`](./skills/agendaforge-webhooks/SKILL.md): Subscribe REST Hook webhooks to 16 trigger events, verify HMAC-SHA256 signed deliveries, pull trigger samples, and unsubscribe cleanly.
- [`agendaforge-integrations`](./skills/agendaforge-integrations/SKILL.md): Connect AgendaForge to Zapier, Make.com, or n8n with an organization API key: the live Zapier app, Make HTTP modules, or the n8n community node.

## Public resources

- [Developer portal](https://agendaforge.app/developers)
- [REST API reference](https://agendaforge.app/docs/api)
- [Authentication guide](https://agendaforge.app/auth.md)
- [Agent index](https://agendaforge.app/llms.txt)
- [Full agent content](https://agendaforge.app/llms-full.txt)
- [OpenAPI 3.1 specification](https://agendaforge.app/openapi.json)
- [AgendaForge MCP](https://mcp.agendaforge.app/mcp)
- [Read-only AgendaForge MCP](https://mcp.agendaforge.app/mcp/readonly)

