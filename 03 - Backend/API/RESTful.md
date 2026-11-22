---
flashcards:
  q-8k3c: { nid: 1795356733443, hash: uav5ff42, sync: r9jycmdr }
  q-ag4n: { nid: 1795356733477, hash: t3usqzws, sync: 6muesdbv }
  q-8ah9: { nid: 1795356733492, hash: 44fygq6k, sync: aszqka8y }
---

# RESTful API Design

**RESTful** describes a concrete web [[API]] that adheres to [[REST]] principles—in industry practice, typically **Richardson Maturity Model Level 2** (resource-oriented [[URI|URIs]] + semantic [[HTTP]] verbs and status codes).

## Practical RESTful Conventions

- **Noun-based, plural resource URIs:** `/v1/orders`, `/v1/orders/{orderId}/items` (avoid verbs like `/getOrder` or `/deleteUser`).
- **Accurate HTTP status codes:**
  - `200 OK`, `201 Created` (with `Location` header), `202 Accepted` (async processing), `204 No Content`.
  - `400 Bad Request` / `422 Unprocessable Entity` (validation), `401 Unauthorized` (missing/invalid [[Authentication]]), `403 Forbidden` (failed [[Authorization]]), `404 Not Found`, `409 Conflict` (duplicate / [[Optimistic Locking]] failure), `429 Too Many Requests` ([[Rate Limiting]]).
  - `500 Internal Server Error`, `502/503/504` (upstream/availability failures).
- **Consistent error envelope:** Use a standard machine-readable format such as **RFC 9457 Problem Details** (`type`, `title`, `status`, `detail`, `instance`, plus domain error code) so clients never parse human error strings.
- **Extensible collection responses:** Wrap lists in an object (`{ "items": [...], "nextCursor": "..." }`) rather than returning a raw JSON array.
- **Stateless auth & concurrency control:** Authenticate via bearer [[JWT]] / [[OIDC]] headers; use `ETag` + `If-Match` headers for HTTP-level [[Optimistic Concurrency Control]].

## Pragmatic Deviations in Production

Strict CRUD mapping does not fit every domain workflow cleanly:
- **State transitions / non-CRUD commands:** Model as sub-resource actions (`POST /orders/{id}/cancel`) or an explicit sub-resource (`POST /orders/{id}/cancellations`).
- **Long-running operations:** Return `202 Accepted` with a status URL (`GET /jobs/{jobId}`) rather than holding an HTTP connection open ([[Client polling pattern]] or [[Webhook]]).

> [!tip] Production reality
> - **401 vs. 403:** Return `401` when identity is missing or expired (client can re-authenticate); return `403` when identity is valid but lacks permission (re-authenticating will not help).
> - **Concurrency conflicts via `ETag` + `If-Match`:** Returning `412 Precondition Failed` (or `409 Conflict`) prevents lost updates in RESTful APIs without holding server-side locks.

## Flashcards

What is the difference between HTTP `401 Unauthorized` and `403 Forbidden` in a RESTful API?::`401` means the caller is unauthenticated (missing/invalid credentials), whereas `403` means the caller is authenticated but lacks [[Authorization|permission]] for the requested resource.
^q-8k3c

How can a RESTful API implement HTTP-level [[Optimistic Concurrency Control]] to prevent lost updates?::Return an `ETag` (version token) on `GET` and require clients to send `If-Match: <etag>` on `PUT`/`PATCH`; reject stale writes with `412 Precondition Failed`.
^q-ag4n

Model long-running backend workflows with `202 Accepted` plus a pollable status resource (or [[Webhook]]) instead of blocking synchronous HTTP requests. #card-reminder
^q-8ah9

## Related
- [[REST]]
- [[API]]
- [[HTTP]]
- [[API versioning]]
- [[API Idempotency]]
- [[Client polling pattern]]
- [[Optimistic Concurrency Control]]
- [[Rate Limiting]]
