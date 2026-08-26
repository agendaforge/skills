---
name: agendaforge-mcp
description: Connect an AI agent to AgendaForge MCP for event reads and human-approved write proposals.
---

# Use AgendaForge MCP

## Connect

Configure Claude, Cursor, ChatGPT, or another Streamable HTTP MCP client with one of these URLs:

```text
https://mcp.agendaforge.app/mcp
https://mcp.agendaforge.app/mcp/readonly
```

Use `/mcp` when the agent needs read tools and approval-gated write tools. Use `/mcp/readonly` when the connection must expose only reads.

## Authenticate

Prefer OAuth 2.1 for a user-authorized connection. Let the client follow the `401` Bearer challenge to `https://mcp.agendaforge.app/.well-known/oauth-protected-resource`, discover the WorkOS AuthKit authorization server, and complete authorization code with PKCE.

Use an organization API key when the client supports API key authentication. An Owner or Admin creates the `afk_live_...` key in AgendaForge. Give it `mcp:read` for read tools and `mcp:write` for approval-gated write tools. Pick only the scopes the client needs.

## Start with reads

Call concrete tools based on the task:

- Organization and event context: `list_my_orgs`, `list_events`, `get_event`
- Program and schedule: `list_sessions`, `get_session`, `get_agenda`, `check_schedule_conflicts`
- People and registration: `list_speakers`, `get_contact`, `search_contacts`, `list_attendees`
- Forms and review: `list_forms`, `list_submissions`, `get_review_progress`
- Email context: `get_email_activity`, `list_email_templates`, `draft_email`

Read tools execute directly. Write and communication tools create proposals. A human reviews and approves each proposal in AgendaForge. For example, use `update_session`, `schedule_session`, `send_speaker_message`, or `create_event` to propose work, then tell the user that approval is still required.

## Request a media upload

Call `request_media_upload` to propose a one-time upload link for an event logo or cover. The request follows the same approval-gated write model, so a human must approve the proposal in AgendaForge before the upload proceeds. Do not describe the media as uploaded merely because the proposal was created.

## Handle failures

On `401`, inspect the `WWW-Authenticate` Bearer challenge and follow its `resource_metadata` URL. Do not guess OAuth endpoints or scopes. On `403`, request the missing scope or access. On `429`, wait for the number of seconds in `Retry-After` before retrying.
