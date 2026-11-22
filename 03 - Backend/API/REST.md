---
flashcards:
  q-ca6t: { nid: 1795356733369, hash: 6n8bg5fd, sync: g5hwzezk }
  q-7tgi: { nid: 1795356733392, hash: 3tpvtshj, sync: dv322qth }
  q-266v: { nid: 1795356733417, hash: xp29q4ta, sync: 96gcctse }
---

# REST (Representational State Transfer)

**REST** is an architectural style for distributed hypermedia systems (defined by Roy Fielding in 2000), most commonly applied over [[HTTP]]. Instead of exposing remote procedures, a REST system models domain entities as **resources** addressed by [[URI|URIs]] and manipulated through a uniform set of operations.

## The 6 Architectural Constraints

1. **Client-Server:** Separates user-interface concerns from data-storage/business-logic concerns so each can evolve independently.
2. **Stateless:** Every request from client to server must contain all context needed to understand and process it (no server-side session state). Enables trivial [[Horizontal scaling]] behind a [[Load Balancing|load balancer]].
3. **Cacheable:** Responses must explicitly declare themselves cacheable or non-cacheable (`Cache-Control`, `ETag`, `Last-Modified`) so clients and intermediaries ([[CDN]], [[Reverse proxy]]) can reuse them safely.
4. **Uniform Interface:** Standardized resource identification ([[URI]]), manipulation through representations (JSON, XML), self-descriptive messages (HTTP verbs, status codes, media types), and hypermedia controls (HATEOAS).
5. **Layered System:** Clients cannot tell whether they are talking directly to the origin server or an intermediary ([[API Gateway]], [[Load Balancing|load balancer]], [[CDN]]).
6. **Code on Demand (optional):** Servers may temporarily extend client functionality by transferring executable code.

## HTTP Method Semantics

| Method | Purpose | Safe (no side effects)? | Idempotent ([[API Idempotency]])? |
| --- | --- | --- | --- |
| `GET` | Read representation | Yes | Yes |
| `HEAD` | Read headers only | Yes | Yes |
| `PUT` | Full replace / upsert at URI | No | Yes |
| `DELETE` | Remove resource | No | Yes |
| `POST` | Create sub-resource / trigger action | No | **No** (unless protected by [[Idempotency Key]]) |
| `PATCH` | Partial update | No | Depends (atomic set = yes; increment = no) |

## Richardson Maturity Model

- **Level 0:** Single endpoint tunneling RPC calls over HTTP `POST`.
- **Level 1:** Distinct resource [[URI|URIs]], still mostly `POST`.
- **Level 2:** Resource URIs + proper HTTP verbs and status codes (where most production APIs live; see [[RESTful]]).
- **Level 3:** Hypermedia controls (HATEOAS) driving state transitions via links in responses.

> [!warning] Failure modes & trade-offs
> - **Over-fetching & under-fetching:** Fixed resource representations force clients to either pull unused fields or make N+1 follow-up HTTP calls (where [[GraphQL]] or [[BFF]] often fits better).
> - **Chatty internal calls & JSON overhead:** For high-throughput internal service-to-service communication, text JSON over REST burns CPU/bandwidth compared to [[gRPC]] + [[Protocol Buffers]].
> - **Non-idempotent `POST` under network timeouts:** If a client times out after sending `POST /payments` and retries, it can charge twice unless the server enforces an [[Idempotency Key]].

## Flashcards

What are the five mandatory architectural constraints of REST (excluding optional Code on Demand)?::Client-Server, Stateless, Cacheable, Uniform Interface, and Layered System.
^q-ca6t

Among standard HTTP methods, `GET`, `PUT`, and `DELETE` are ==idempotent==, whereas `POST` is ==not idempotent== by default (and requires an [[Idempotency Key]] for safe retries).
^q-7tgi

## Why does REST's Stateless constraint simplify horizontal scaling? #card
Because the server stores no client session state between requests, any request can be routed to any backend replica behind a [[Load Balancing|load balancer]] without sticky sessions or session replication.
^q-266v

## Related
- [[RESTful]]
- [[API]]
- [[HTTP]]
- [[API Idempotency]]
- [[API versioning]]
- [[API Performance improvement]]
- [[gRPC]]
- [[GraphQL]]
- [[URI]]
