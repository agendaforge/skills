---
name: agendaforge-api
description: Integrate an AI agent with AgendaForge REST API v1, REST Hooks, and create actions.
---

# Use AgendaForge REST API v1

## When to use this skill

Use this skill for server-to-server integrations, scripts, and automations that need JSON over HTTP instead of MCP. Best-fit jobs: validating a key, subscribing REST Hook webhooks to trigger events, pulling trigger samples, and creating contacts, sessions, and sponsors with safe idempotent retries. Authenticate every request with an organization API key that carries the `public_api` scope. Do not use it for interactive AI-client sessions, which the `agendaforge-mcp` skill is built for, or for public product questions, which need no key at all (the `agendaforge-docs` skill).

## Get an API key

Ask the user (or their AgendaForge Owner or Admin) to create the key in the dashboard:

1. Sign in at `https://dash.agendaforge.app` and open the organization.
2. Go to **Organization Settings, API tokens** (`/{org-slug}/settings/api-tokens`). This page requires admin access.
3. Create a token and select scopes: **Public REST API** (`public_api`) for this skill's endpoints; add `mcp:read` or `mcp:write` only if the same key will also authenticate an MCP connection.
4. Copy the `afk_live_...` value immediately. It is shown once; AgendaForge stores only a hash and cannot show it again.

Treat the key like a password: it acts as the organization member who created it.

## Test the connection first

Send all requests to `https://api.agendaforge.app/api/v1`. Authenticate with an `afk_live_...` organization API key that has the `public_api` scope. Use either `Authorization: Bearer` or `X-API-Key`.

```sh
curl https://api.agendaforge.app/api/v1/me \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

Expect an object with `orgId` and `orgName`. Stop and resolve authentication if this request fails.

## Subscribe and unsubscribe REST Hooks

Subscribe an HTTP or HTTPS target URL to a documented trigger event:

```sh
curl -X POST https://api.agendaforge.app/api/v1/hooks \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{"event":"contact.added","targetUrl":"https://example.com/hooks/agendaforge"}'
```

Save the returned `id`, then use it to unsubscribe:

```sh
curl -X DELETE https://api.agendaforge.app/api/v1/hooks/HOOK_ID \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

Re-subscribing the same event and URL returns the existing subscription. Deleting an unknown or already removed subscription still succeeds.

## Create a contact

```sh
curl -X POST https://api.agendaforge.app/api/v1/contacts \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: contact-ada-001" \
  -d '{
    "firstName":"Ada",
    "lastName":"Lovelace",
    "email":"ada@example.com",
    "type":"speaker",
    "company":"Analytical Engines"
  }'
```

## Create a session

```sh
curl -X POST https://api.agendaforge.app/api/v1/sessions \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: session-opening-001" \
  -d '{
    "eventId":"EVENT_ID",
    "title":"Opening keynote",
    "type":"keynote",
    "duration":45,
    "isPublic":true,
    "tags":["opening"]
  }'
```

## Create a sponsor

```sh
curl -X POST https://api.agendaforge.app/api/v1/sponsors \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: sponsor-globex-001" \
  -d '{
    "eventId":"EVENT_ID",
    "name":"Globex Corporation",
    "website":"https://globex.example.com",
    "description":"Event sponsor",
    "hasBooth":true,
    "tierId":"TIER_ID"
  }'
```

## List, batch, and async import

Read records back with the cursor-paginated list endpoints: `GET /contacts` (the organization's contacts), and `GET /sessions?eventId=...` and `GET /sponsors?eventId=...` (one owned event; `eventId` is required, and an event owned by another organization returns `403 FORBIDDEN`). All three return `{ "data": [...], "hasMore": boolean, "nextCursor": string|null }`, newest first. Page with `limit` (1-100, default 25) and follow `nextCursor` until `hasMore` is `false`; cursors are opaque, so never construct one.

```sh
curl "https://api.agendaforge.app/api/v1/contacts?limit=100" \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
# then, while hasMore is true, request the next page with the returned cursor:
curl "https://api.agendaforge.app/api/v1/contacts?limit=100&cursor=NEXT_CURSOR" \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

Create up to 100 contacts in one synchronous call with `POST /contacts/batch`. Items succeed or fail independently: the `200` response pairs every input `index` with either the created contact or a typed error, plus a `summary` of `total`, `created`, and `failed`. Each item behaves exactly like a single `POST /contacts`, including firing the `contact.added` trigger.

```sh
curl -X POST https://api.agendaforge.app/api/v1/contacts/batch \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: batch-attendees-001" \
  -d '{"contacts":[{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com"},{"firstName":"Grace","lastName":"Hopper","email":"grace@example.com"}]}'
```

For up to 1,000 contacts, use the async import. `POST /imports/contacts` answers `202 Accepted` with `{ jobId, type: "contacts_import", status: "pending", total, statusUrl }` and a `Location` header pointing at the poll URL. Poll `GET /jobs/{id}` with backoff until `status` is `completed` or `failed`; the job `result` carries `total`, `processed`, `created`, `failed`, and the first 50 per-item errors. Jobs are retained for 7 days; unknown, expired, and cross-organization ids all answer `404`.

```sh
curl -X POST https://api.agendaforge.app/api/v1/imports/contacts \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: import-attendees-001" \
  -d '{"contacts":[{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com"}]}'

curl https://api.agendaforge.app/api/v1/jobs/JOB_ID \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

Both endpoints accept an `Idempotency-Key` header.

## Retry creates safely

Send an `Idempotency-Key` on create requests. AgendaForge retains it for 24 hours. Repeating the same request on the same endpoint returns the original response with `Idempotent-Replay: true`. Reusing the key on a different endpoint returns `409 CONFLICT`, so generate a new key for a different operation.

## Parse errors and rate limits

Handle every non-2xx response through the shared JSON shape:

```json
{
  "error": {
    "code": "UNAUTHENTICATED",
    "message": "Missing API key. Send 'Authorization: Bearer <key>'."
  }
}
```

Branch on `error.code` and surface `error.message`. On `429 RATE_LIMITED`, wait for `Retry-After` seconds before retrying.

## Use embed feeds for published programs

AgendaForge provides embeddable widgets and public JSON feeds for published sessions, speakers, and agendas. Use those feeds for a published program display. Use REST API v1 for the documented authenticated integration operations above.
