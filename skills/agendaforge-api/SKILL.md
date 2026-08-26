---
name: agendaforge-api
description: Integrate an AI agent with AgendaForge REST API v1, REST Hooks, and create actions.
---

# Use AgendaForge REST API v1

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
