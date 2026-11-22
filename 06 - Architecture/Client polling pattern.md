In the **client polling pattern**, after the client initiates a long-running request, it repeatedly checks ("polls") the server for the status of the request until it reaches a terminal state (`COMPLETED` or `FAILED`). This pattern is useful in [[Serverless|serverless]] and asynchronous microservice architectures (e.g., waiting for a [[Saga]] or background job to finish) where processing takes longer than a synchronous [[HTTP]] request timeout.

## Standard HTTP Flow (Async Request-Reply)

1. **Submit**: `POST /reports` → Server validates, enqueues work (e.g., via [[Outbox Pattern]] / [[Kafka]]), and immediately returns `202 Accepted` with a status URL (`Location: /reports/req-42/status`) and optional `Retry-After: 2`.
2. **Poll**: Client periodically sends `GET /reports/req-42/status` → returns `200 OK` with `{ "status": "PENDING" }` (or `303 See Other` pointing to the final resource once done).
3. **Complete**: Client receives `{ "status": "COMPLETED", "resultUrl": "..." }` and stops polling.

## Short Polling vs. Long Polling vs. Push Alternatives

| Pattern | How It Works | Pros | Cons |
| --- | --- | --- | --- |
| **Short Polling** | Client sends periodic `GET` every \(T\) seconds | Stateless, trivial to cache/scale, firewall-friendly | Wasted empty requests; latency bounded by poll interval |
| **Long Polling** | Server holds the `GET` open until state changes or timeout expires | Near-real-time updates over plain [[HTTP]] | Ties up server connections/threads; proxy timeouts |
| **Server-Sent Events (SSE)** | Unidirectional HTTP stream from server to client | Simple, auto-reconnect, lightweight for server-to-client feeds | Unidirectional only; connection limits on [[HTTP1.1]] |
| **[[WebSocket]]** | Full-duplex persistent [[TCP]] connection | Lowest latency, bidirectional | Stateful connections; harder [[Load Balancing]] & failover |
| **[[Webhook]]** | Server calls a client-provided callback URL on completion | Zero client polling load; ideal for server-to-server | Client must expose a public endpoint + handle retries/auth |

> [!warning] Failure modes & trade-offs
> - **Polling storms (thundering herd)**: If 50,000 clients poll every 1s during a downstream slowdown, polling traffic itself DDoSes your status endpoint. **Mitigate** with exponential backoff + jitter, honoring server-controlled `Retry-After` headers, serving status from a fast read store/[[Redis]] cache, and enforcing [[Rate Limiting]].
> - **Stuck forever**: Always give clients a maximum poll duration / timeout and ensure the backend transitions stalled jobs to `FAILED` / `EXPIRED` (via [[Reconciliation]] or [[Worker Leases]]).

## How do you prevent Client Polling from turning into a self-inflicted DDoS during a backend slowdown? #card
1. Return a server-controlled **`Retry-After`** header so the backend can dynamically throttle poll frequency under load.
2. Use **exponential backoff with jitter** and a hard client-side max poll duration.
3. Serve status reads from a lightweight cache/read projection ([[CQRS]] / [[Redis]]) rather than the primary write DB.

In the Async Request-Reply pattern, the server acknowledges accepted background work with HTTP ==202 Accepted== and a ==Location== status endpoint for the client to poll.

## Related
- [[WebSocket]]
- [[Webhook]]
- [[HTTP]]
- [[REST]]
- [[Serverless]]
- [[Saga]]
- [[Rate Limiting]]
- [[Retry Pattern]]
