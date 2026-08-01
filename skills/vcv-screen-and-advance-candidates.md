---
generated: '2026-07-21'
method: generated
name: Screen candidate responses and advance them through the funnel
description: >-
  List a vacancy's candidate responses, review video-interview answers and
  comments, then move a response to the next hiring status.
api: openapi/vcv-openapi.yml
operations:
  - 'GET /api/v3/vacancies'
  - 'GET /api/v3/responses'
  - 'GET /api/v3/responses/{id}'
  - 'GET /api/v3/videointerviews/response'
  - 'GET /api/v3/response-statuses'
  - 'PATCH /api/v3/responses/{id}'
  - 'POST /api/v3/response-comments'
source: >-
  Grounded in openapi/vcv-openapi.yml (developer.vcv.ru swagger.yml). The spec
  declares no operationIds, so operations are anchored by METHOD + path,
  verified verbatim in the bundled spec.
---

# Screen candidate responses and advance them through the funnel

## Auth
- Every request sends `Authorization: Bearer <token>` (the only security scheme; see `authentication/vcv-authentication.yml`). Base URL `https://my.vcv.ai` (or `https://my.vcv.ru`).

## Conventions
- Lists paginate with `page` + `limit` and return HAL envelopes: read results from `_embedded`, totals from `_total_items`, next page from `_links.next.href`. Filter with `filter[...]` params; embed relations with `with[]`. See `conventions/vcv-conventions.yml`.
- There is **no idempotency contract** — do not blind-retry the PATCH/POST steps; re-read state first. Errors are largely undocumented (`errors/vcv-problem-types.yml`), so treat any non-2xx as terminal and inspect the body.

## Steps
1. **Find the vacancy** — `GET /api/v3/vacancies` with `filter[active]=1` (paginate with `page`/`limit`). Capture the vacancy `id`.
2. **List its responses** — `GET /api/v3/responses` with `filter[vacancy_id]=<id>`, optionally `filter[response_status_id]=<status>` to work one funnel stage. Read `_embedded`.
3. **Open a response** — `GET /api/v3/responses/{id}` with `with[]` relations to embed the candidate and interview context.
4. **Review video answers** — `GET /api/v3/videointerviews/response` with `filter[response_id]=<id>` to fetch the candidate's video-interview submission for that response.
5. **Pick the target status** — `GET /api/v3/response-statuses` and choose the next `id` in the company's funnel (statuses are company-defined, tree-shaped via `parent_id`).
6. **Advance the response** — `PATCH /api/v3/responses/{id}` with body `{"response_status_id": <target>}`. If a `response_status_changed` webhook is registered, downstream systems are notified (see `asyncapi/vcv-webhooks-asyncapi.yml`).
7. **Leave a review note (optional)** — `POST /api/v3/response-comments` with the response id and comment text.
