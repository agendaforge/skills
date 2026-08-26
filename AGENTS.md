# Integrate AgendaForge

Start with the machine-readable OpenAPI 3.1 specification at https://agendaforge.app/openapi.json. Send REST requests to `https://api.agendaforge.app/api/v1`.

Authenticate REST requests with an organization API key created by an Owner or Admin. Keys begin with `afk_live_` and can be sent as `Authorization: Bearer afk_live_...` or `X-API-Key: afk_live_...`. Authenticate an MCP connection with OAuth 2.1 through WorkOS AuthKit, or use an organization API key with the required MCP scope. Follow the OAuth discovery metadata and use authorization code with PKCE instead of guessing authorization endpoints or scopes.

Read public content in Markdown by appending `.md` to any AgendaForge page URL. Start agent discovery at https://agendaforge.app/llms.txt and use https://agendaforge.app/llms-full.txt for the full content collection.

Use the repository skills for task-specific instructions:

- [`agendaforge-mcp`](./agendaforge-mcp/SKILL.md) for MCP clients and tools.
- [`agendaforge-api`](./agendaforge-api/SKILL.md) for REST API integrations.

