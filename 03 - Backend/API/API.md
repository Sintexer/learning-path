---
flashcards:
  q-w3t4: { nid: 1795356733526, hash: nvb8smg3, sync: ukfsrpse }
  q-q5gs: { nid: 1795356733546, hash: par55kqc, sync: w2pamhru }
---

APIs are mechanisms that facilitate two software components communicating with each other. APIs act as the front door for applications to access data, business logic, or functionality from backend services.

## Common Backend API Styles

| Style | Transport / Encoding | Best For | Key Trade-off |
| --- | --- | --- | --- |
| [[REST]] / [[RESTful]] | [[HTTP]] + JSON | Public APIs, CRUD resources, cacheable reads | Over-/under-fetching, verbose JSON, weaker schema enforcement |
| [[gRPC]] | [[HTTP2]] + [[Protocol Buffers]] | Internal service-to-service, low-latency, streaming | Not human-readable, weaker browser support without proxy |
| [[GraphQL]] | [[HTTP]] + typed schema | Complex UIs aggregating many entities ([[BFF]]) | Caching is harder, N+1 resolver risk, query-cost DoS |
| [[WebSocket]] / [[Webhook]] | Persistent TCP / async HTTP callbacks | Real-time push or event notifications | Stateful connections (WebSocket) or retry/delivery guarantees (Webhook) |

## Core API Design Concerns

- **Evolution & Contracts:** Maintain [[Backward Compatibility]] and [[Forward Compatibility]]; choose an [[API versioning]] strategy before publishing external contracts.
- **Safety under Retries:** Make mutating endpoints safe to retry using [[API Idempotency]] and an [[Idempotency Key]].
- **Protection & Resilience:** Combine [[Authentication]]/[[Authorization]], [[Rate Limiting]], [[Timeout Pattern|timeouts]], [[Circuit Breaker|circuit breakers]], and [[Backpressure]] (often at an [[API Gateway]]).
- **Performance:** Pagination, caching, compression, and connection pooling (see [[API Performance improvement]]).

> [!warning] Failure modes & trade-offs
> - **Leaking internal models:** Exposing database entities directly as API payloads couples external clients to internal schema refactors—always use dedicated request/response DTOs.
> - **Unbounded responses:** Returning raw arrays without an envelope or pagination breaks forward extensibility and risks OOM/timeouts as data grows.
> - **Breaking changes without deprecation windows:** Removing fields or changing semantics silently breaks mobile/third-party clients that cannot upgrade simultaneously.

## Flashcards

## When should you choose [[gRPC]] over [[REST]] for a backend API? #card
Choose [[gRPC]] for internal service-to-service calls where low latency, strict typed contracts ([[Protocol Buffers]]), binary serialization efficiency, or bidirectional streaming over [[HTTP2]] matter. Choose [[REST]] for public/browser-facing APIs, HTTP caching, and broad tooling compatibility.

Why should REST API list endpoints return a wrapper object (e.g. `{ "items": [...], "nextCursor": "..." }`) instead of a bare JSON array?::A bare array root cannot be extended with metadata (pagination cursors, totals, warnings) without making a breaking change to the response schema.
^q-w3t4

Always pair client [[Retry Pattern|retries]] on mutating API endpoints with [[API Idempotency]] (e.g. an [[Idempotency Key]]) to prevent duplicate side effects. #card-reminder
^q-q5gs

## Related
- [[REST]]
- [[RESTful]]
- [[gRPC]]
- [[GraphQL]]
- [[API versioning]]
- [[API Performance improvement]]
- [[API Idempotency]]
- [[API Gateway]]
- [[Rate Limiting]]
- [[Backward Compatibility]]