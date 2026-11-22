---
flashcards:
  q-788k: { nid: 1795356719891, hash: xmmc2x6i, sync: vxd3e9de }
  q-ntvq: { nid: 1795361710380, hash: 659eifsi, sync: msp9ksyk }
  q-j645: { nid: 1795361710453, hash: fskxgzkd, sync: 4i2gbkwh }
  q-qh7b: { nid: 1795356720017, hash: sgwr8ss5, sync: rmapvdk6 }
---

**GraphQL** is a data query and manipulation language for [[API|APIs]], and a runtime for executing those queries against your existing data sources. It was developed by Facebook in 2012 and later open-sourced.

- **Efficiency (No Over-/Under-Fetching):** Unlike traditional [[REST|REST APIs]], the client specifies the exact fields it needs, reducing payload size over mobile or high-latency links.
- **Single Request:** The client can fetch an entire object graph in a single round trip instead of chaining multiple REST calls (solving classic [[BFF]] / [[Aggregator]] chatty-client problems).
- **Strong Typing:** Every GraphQL schema is strongly typed (queries, mutations, subscriptions, input types), enabling introspection, codegen, and clear contracts.
- **Real-Time Updates:** Implemented with **subscriptions** (typically transported over [[WebSocket]] or Server-Sent Events).
- **Integration & Federation:** Wraps existing [[REST]], [[gRPC]], or [[Database]] backends behind a unified graph (or composes subgraphs across teams via Apollo Federation).

> [!warning] Failure modes & trade-offs
> - **N+1 query problem**: naive field resolvers fire a separate DB/RPC call for every item in a list—always batch and deduplicate per-request using **DataLoader**.
> - **Lost [[HTTP]] caching**: because GraphQL typically sends `POST /graphql` with queries in the body, standard [[CDN]] and browser HTTP caching stops working unless you use **Persisted Queries** over `GET`.
> - **Query-depth / complexity DoS**: a malicious or buggy client can craft deeply nested cyclic queries (`user { friends { friends { ... } } }`) that exhaust server CPU/memory—enforce **query depth limits**, **cost-based [[Rate Limiting]]**, and **persisted query allowlists** in production.
> - **HTTP 200 for errors**: GraphQL returns `200 OK` even with partial/full `errors[]` in the JSON body, which blinds naive L7 monitoring and [[Circuit Breaker|circuit breakers]] unless you parse GraphQL-level error rates.

## Flashcards

GraphQL::A strongly typed API query language and runtime where clients declare the exact shape of data they need in a single request (queries, mutations, subscriptions).
^q-788k

How do you solve the N+1 resolver problem in GraphQL?::Use a per-request batching and caching layer like ==DataLoader== to coalesce individual field fetches into batched DB/service queries.
^q-b6yz
^q-ntvq

Why does GraphQL break standard [[HTTP]] / [[CDN]] caching by default, and how do you regain it?::GraphQL sends queries in `POST` bodies to a single `/graphql` endpoint; you regain HTTP caching via ==Automatic Persisted Queries (APQ)== sent as `GET` requests with a query hash.
^q-mphb
^q-j645

### What security controls are mandatory before exposing a public GraphQL API? #card

- **Query depth & complexity limits** (to block cyclic nested DoS queries).
- **Persisted query allowlists** (clients send pre-approved query hashes instead of arbitrary ASTs).
- **Disable introspection** in production (if the schema is private) and enforce field-level [[Authorization]].
^q-qh7b

## Related
- [[REST]]
- [[gRPC]]
- [[WebSocket]]
- [[BFF]]
- [[Aggregator]]
- [[API]]
- [[Rate Limiting]]
