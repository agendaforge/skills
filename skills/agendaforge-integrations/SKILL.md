---
name: agendaforge-integrations
description: Connect AgendaForge to Zapier, Make.com, or n8n with an organization API key and REST Hook triggers.
---

# Connect AgendaForge to Zapier, Make.com, or n8n

## When to use this skill

Use this skill when helping a user wire AgendaForge into a no-code or low-code automation platform: Zapier, Make.com, or n8n. Best-fit jobs: fanning contact, sponsor, and session activity out to other apps the moment it happens, and pushing records from a form, CRM, or spreadsheet back into AgendaForge. All three platforms authenticate with the same organization API key and run on the same REST API v1, so a workflow modeled on one can be modeled on the others. For custom server-to-server code, use the `agendaforge-api` skill directly; for interactive AI-client sessions, use the `agendaforge-mcp` skill.

## Create the API key

Ask the user (or their AgendaForge Owner or Admin) to create the key in the dashboard:

1. Sign in at `https://dash.agendaforge.app` and open the organization.
2. Go to **Organization Settings, API tokens** (`/{org-slug}/settings/api-tokens`). This page requires admin access.
3. Create a token with the **Public REST API** (`public_api`) scope.
4. Copy the `afk_live_...` value immediately. It is shown once; AgendaForge stores only a hash.

Keys are organization-wide and shared across Zapier, Make.com, and n8n: revoking one disconnects all of them, so rotate deliberately.

## Zapier

AgendaForge has a live app on Zapier. Open the listing at `https://zapier.com/apps/agendaforge/integrations`, pair it with another app, and connect the AgendaForge account by pasting the `afk_live_...` key when prompted. The integration offers instant webhook-style triggers for contact, sponsor, and session events (added, updated, removed) and create actions that write contacts, sessions, and sponsors back into AgendaForge. Pull a trigger sample, map fields, test the Zap, and turn it on.

## Make.com

There is no published one-click Make app: connect through the REST API and REST Hook webhooks with Make's HTTP and webhook modules. Subscribe a scenario's webhook URL to a trigger event, tagging the subscription for Make:

```sh
curl -X POST "https://api.agendaforge.app/api/v1/hooks?source=make" \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{"event":"contact.added","targetUrl":"MAKE_WEBHOOK_URL"}'
```

To write back, point a Make HTTP module at `POST /api/v1/contacts`, `POST /api/v1/sessions`, or `POST /api/v1/sponsors` with the same key. Unsubscribe with `DELETE /api/v1/hooks/{id}`.

## n8n

Two ways to connect, both using the same key and endpoints:

1. **Community node (recommended):** install the npm package `n8n-nodes-agendaforge` (Settings, Community Nodes, Install on self-hosted n8n; on n8n Cloud it appears in the node panel once verified). Create an AgendaForge API credential with the `afk_live_...` key; it is validated against `GET /api/v1/me`. The AgendaForge node creates Contacts, Sessions, and Sponsors (records are stamped `source: "n8n"`), and the AgendaForge Trigger node auto-subscribes its webhook when the workflow is activated and unsubscribes on deactivation.
2. **Generic HTTP Request node (no install):** call the REST API directly with a Bearer header for create actions. For triggers, subscribe an n8n Webhook node's production URL manually with `POST /api/v1/hooks?source=n8n` and a body of `{"event":"contact.added","targetUrl":"..."}`, and unsubscribe with `DELETE /api/v1/hooks/{id}`.

## Read more

- REST API reference: `https://agendaforge.app/docs/api`
- n8n guide: `https://agendaforge.app/docs/n8n`
- Zapier and Make.com guide: `https://agendaforge.app/integrates-with/zapier`
- All integrations: `https://agendaforge.app/integrations`
