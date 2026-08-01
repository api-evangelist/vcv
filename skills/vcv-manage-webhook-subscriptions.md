---
generated: '2026-07-21'
method: generated
name: Subscribe to VCV recruitment events via webhooks
description: >-
  Register, verify, update, and remove company webhook subscriptions for the
  five VCV event types.
api: openapi/vcv-openapi.yml
operations:
  - 'POST /api/v3/company-webhooks'
  - 'GET /api/v3/company-webhooks'
  - 'GET /api/v3/company-webhooks/{id}'
  - 'PATCH /api/v3/company-webhooks/{id}'
  - 'DELETE /api/v3/company-webhooks/{id}'
source: >-
  Grounded in openapi/vcv-openapi.yml (developer.vcv.ru swagger.yml). The spec
  declares no operationIds, so operations are anchored by METHOD + path,
  verified verbatim in the bundled spec.
---

# Subscribe to VCV recruitment events via webhooks

## Auth
- `Authorization: Bearer <token>` on every request (see `authentication/vcv-authentication.yml`). Base URL `https://my.vcv.ai`.

## Event catalog
Exactly five event types are accepted (enum on the create body — see `asyncapi/vcv-webhooks-asyncapi.yml`):
`response_created`, `response_status_changed`, `response_comment_created`, `candidate_refused`, `vacancy_created`.
One subscription carries one event type; register multiple subscriptions to cover several events. Payload schemas are not published — log raw deliveries before parsing.

## Steps
1. **Create the subscription** — `POST /api/v3/company-webhooks` with body `{"event": "<one of the five>", "url": "https://your-listener.example/hook", "secret": "<your shared secret>", "name": "<label>", "active": true}`. The `secret` is stored per subscription; use it to authenticate deliveries at your endpoint.
2. **Verify it landed** — `GET /api/v3/company-webhooks` with `filter[event]=<event>` or `filter[name]=<label>`; results arrive in the HAL `_embedded` envelope (`conventions/vcv-conventions.yml`). Or fetch it directly with `GET /api/v3/company-webhooks/{id}`.
3. **Pause or repoint** — `PATCH /api/v3/company-webhooks/{id}` with `{"active": false}` to pause, or a new `url`/`secret`/`name`. Note the event type itself is not patchable — create a new subscription to change events.
4. **Remove** — `DELETE /api/v3/company-webhooks/{id}` when the listener is retired.

## Cautions
- No idempotency keys: a retried create POST makes a duplicate subscription — list-and-check before re-creating.
- Rotate `secret` via PATCH periodically; deliveries have no other documented signing mechanism.
