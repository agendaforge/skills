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

Use `/mcp` when the agent needs read tools and approval-gated write tools. Use `/mcp/readonly` when the connection must expose only reads. A separate no-auth docs server at `https://mcp.agendaforge.app/mcp/docs` searches public documentation content only.

## Authenticate

Prefer OAuth 2.1 for a user-authorized connection. Let the client follow the `401` Bearer challenge to `https://mcp.agendaforge.app/.well-known/oauth-protected-resource`, discover the WorkOS AuthKit authorization server, and complete authorization code with PKCE.

Or use an organization API key: an Owner or Admin creates it at `dash.agendaforge.app` under Organization Settings, API tokens (see the `agendaforge-api` skill for the exact steps). Give it `mcp:read` for read tools and `mcp:write` for approval-gated write tools. Pick only the scopes the client needs.

## The write model

Read tools execute directly and return data. Write and communication tools NEVER change data directly: each call creates a proposal that a human reviews and approves inside AgendaForge. After calling a write tool, tell the user the proposal exists and approval is still required. Use `list_pending_approvals` to see what is waiting.

## Read tools (24)

Organization and event context:

- `list_my_orgs`: List organizations available to the authenticated identity.
- `list_events`: List events in an organization with dates, status, and links.
- `get_event`: Get event details, venue, links, and live event counters.

Program and schedule:

- `list_sessions`: List event sessions with optional status, track, and schedule filters.
- `get_session`: Get one session with description, speakers, room, track, and time.
- `get_agenda`: Get the scheduled agenda grouped by day and room.
- `list_tracks_rooms`: List event tracks and rooms with available metadata.
- `check_schedule_conflicts`: Check proposed session placements for conflicts without changing data.

People and CRM:

- `list_speakers`: List event speakers, sessions, confirmation state, and contact ids.
- `get_contact`: Get an event contact profile, status, sessions, and notes count.
- `search_contacts`: Search or page through organization contacts.
- `get_speaker_card_link`: Get the latest public speaker card URL for an event contact.

Search:

- `search`: Search accessible organization records by keyword.
- `semantic_search`: Run metered hybrid vector and text search over accessible records.

Forms and review:

- `list_forms`: List event forms with status and submission counts.
- `list_submissions`: List submissions for a form with optional status filtering.
- `get_review_progress`: Get review rounds, completion percentages, and pending decisions.

Registration:

- `get_registration_stats`: Get registration statistics and the active ticketing provider rail.
- `list_attendees`: List ticketed event attendees with ticket status and type.

Speaker portal and email:

- `list_speaker_tasks`: List event speaker portal tasks and their status.
- `get_email_activity`: List recent event email sends and delivery status.
- `list_email_templates`: List organization and event email template names and ids.
- `draft_email`: Render an email template preview without sending or logging an email.
- `list_pending_approvals`: List pending Coordinator approvals for an event.

## Write tools (27, all approval-gated)

Events, sessions, and schedule:

- `create_event`: Propose creating a draft event for the organization.
- `create_session`: Propose a new unscheduled session in proposed status.
- `update_session`: Propose changes to session content, taxonomy, or schedule.
- `schedule_session`: Propose placing, moving, swapping, or unscheduling a session.
- `delete_session`: Propose permanent session deletion, disabled by default and approval-gated.
- `create_session_from_submission`: Propose creating a session from an approved submission.
- `manage_tracks`: Propose creating or renaming an event track.
- `manage_rooms`: Propose creating or renaming an event room.

Speakers and contacts:

- `assign_speaker`: Propose assigning a contact as a session speaker.
- `remove_session_speaker`: Propose removing a speaker assignment from a session.
- `update_contact`: Propose changes to a contact profile, tags, or custom fields.
- `add_contact_note`: Propose a private internal CRM note for human approval.
- `create_speaker_task`: Propose a new portal task for a speaker.
- `invite_to_portal`: Propose portal access and an invitation email for a contact.

Sponsors:

- `update_sponsor`: Propose changes to sponsor details or custom fields.
- `delete_sponsor`: Propose permanent sponsor deletion, disabled by default and approval-gated.

Forms and review:

- `create_form`: Propose creating a new event form as a draft.
- `update_form`: Propose editing a draft form by instruction.
- `update_form_pipeline`: Propose replacing a form submission pipeline.
- `set_submission_status`: Propose moving a form submission through its pipeline.
- `decide_submission`: Propose accepting or rejecting up to 25 submissions.

Registration:

- `check_in_attendee`: Propose an idempotent attendee check-in by ticket code or id.

Email and messaging:

- `create_email_template`: Propose drafting a new event email template.
- `send_email`: Propose sending a templated email to one contact.
- `send_speaker_message`: Propose an email to a speaker or contact for human approval.
- `send_speaker_reminder`: Propose a speaker task reminder email for human approval.

Media:

- `request_media_upload`: Propose a one-time event logo or cover upload link. A human must approve before the upload proceeds; do not describe the media as uploaded merely because the proposal was created.

## Handle failures

On `401`, inspect the `WWW-Authenticate` Bearer challenge and follow its `resource_metadata` URL. Do not guess OAuth endpoints or scopes. On `403`, request the missing scope or access. On `429`, wait for the number of seconds in `Retry-After` before retrying.
