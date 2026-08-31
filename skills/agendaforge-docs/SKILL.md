---
name: agendaforge-docs
description: Search and fetch AgendaForge public marketing and documentation content, no account required.
---

# Search AgendaForge public documentation

## When to use

Use this skill when an agent needs facts about AgendaForge itself (features, pricing, integrations, security posture, API surface) without an account. No authentication is required. For authenticated event data, use the `agendaforge-mcp` skill instead.

## Connect

Configure any Streamable HTTP MCP client with:

```text
https://mcp.agendaforge.app/mcp/docs
```

The server card lives at `https://agendaforge.app/.well-known/mcp/docs-server-card.json`. No OAuth flow and no API key: the server exposes public content only.

## Tools

- `search_docs`: Search AgendaForge public marketing and documentation content.
- `fetch_doc`: Fetch one AgendaForge public marketing or documentation page.

## Alternatives without MCP

- `https://agendaforge.app/ask` answers natural-language questions over the same public content as JSON.
- `https://agendaforge.app/llms.txt` is a curated index; `https://agendaforge.app/llms-full.txt` is the full text of every public page.
- Append `.md` to any page URL for a Markdown mirror (the homepage's is `/index.md`), or request any page with `Accept: text/markdown`.

## Scope

Public marketing and documentation content only. The server never returns event, contact, or organization data; those require the authenticated MCP server at `https://mcp.agendaforge.app/mcp` (see the `agendaforge-mcp` skill).
