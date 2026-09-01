---
name: agendaforge-webhooks
description: Subscribe to AgendaForge REST Hook webhooks, verify signed deliveries, and handle retries.
---

# Receive AgendaForge webhooks

## When to use this skill

Use this skill to react to AgendaForge activity in real time: subscribe an HTTP endpoint to trigger events, verify each signed delivery, and process contact, sponsor, session, speaker, submission, review, and event changes as they happen. Best-fit jobs: keeping a CRM or spreadsheet in sync, notifying a channel when a submission lands, or driving any downstream automation from event activity. Management calls go to REST API v1 at `https://api.agendaforge.app/api/v1` with an organization API key that has the `public_api` scope (see the `agendaforge-api` skill for creating one). Do not use it to read or create records on demand: that is the `agendaforge-api` skill. For interactive AI-client sessions, use the `agendaforge-mcp` skill.

## Choose a trigger event

These 16 event names are accepted by `POST /api/v1/hooks` and `GET /api/v1/triggers/{event}/sample`:

- `contact.added`, `contact.added_to_event`, `contact.updated`, `contact.removed`
- `sponsor.added`, `sponsor.updated`, `sponsor.removed`
- `session.added`, `session.updated`, `session.removed`
- `speaker.confirmed`, `speaker.declined`
- `submission.created`, `submission.status_changed`
- `review.completed`
- `event.published`

## Subscribe

Send the event name and an http(s) target URL:

```sh
curl -X POST https://api.agendaforge.app/api/v1/hooks \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{"event":"contact.added","targetUrl":"https://example.com/hooks/agendaforge"}'
```

The response is `{ "id": "..." }`. Save the subscription id, and save the endpoint secret shown when you create the subscription: you need it to verify signatures. Re-subscribing the same URL and event is a no-op that returns the existing id. An optional `?source=make` or `?source=n8n` query parameter tags the platform the subscription belongs to; any other value, or omitting it, records `zapier`.

## Map fields with samples

Pull recent records for field mapping before deliveries arrive:

```sh
curl https://api.agendaforge.app/api/v1/triggers/contact.added/sample \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

Each object matches the `payload` field of a live delivery exactly. The array may be empty for a new organization.

## Parse the delivery envelope

Each delivery POSTs an envelope to the target URL; the resource object is nested under `payload`, so map `payload.*` rather than the top level:

```json
{
  "event": "contact.added",
  "timestamp": "2026-06-29T14:03:21.000Z",
  "payload": { "id": "...", "firstName": "Ada" }
}
```

Deliveries also carry headers: `X-Webhook-Event` (same as `event` in the body, for routing without parsing), `X-Webhook-Delivery` (unique delivery id, stable across retries, use it to deduplicate), and `X-Webhook-Signature`.

## Verify the signature

`X-Webhook-Signature` is `sha256=` followed by the hex HMAC-SHA256 of the raw request body, keyed with the endpoint secret. Compute it over the bytes as received, before any JSON parsing, and compare in constant time:

```js
const expected = crypto
  .createHmac("sha256", endpointSecret)
  .update(rawRequestBody)
  .digest("hex");
const received = req.headers["x-webhook-signature"].replace("sha256=", "");
const ok = crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(received));
```

## Respond and handle retries

Respond `2xx` within 10 seconds. A non-2xx response or a timeout is retried up to 3 times with a 1s, 4s, 9s backoff. A `404` or `410` is treated as a permanently dead endpoint: retries stop and the subscription is deactivated. Use `X-Webhook-Delivery` to deduplicate retried deliveries.

## Unsubscribe

```sh
curl -X DELETE https://api.agendaforge.app/api/v1/hooks/HOOK_ID \
  -H "Authorization: Bearer afk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

Idempotent: deleting an unknown or already removed subscription still succeeds.
