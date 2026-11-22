Simple, but people forget to mention it — and it's foundational to everything else.

**The rule:** Never let a remote call wait indefinitely. An unbounded wait means an unbounded hold on a thread, socket, and memory — which is exactly what causes the thread pool exhaustion problem [[Circuit Breaker]] and [[Bulkhead Pattern]] exist to prevent. Timeout is the _first line of defense_; Circuit Breaker is what happens after repeated timeouts.

## Types of Timeouts to Configure

A single "timeout" is rarely enough — configure each stage explicitly:

1. **Connection timeout** — max time to establish the [[TCP]]/[[TLS]] handshake (short, e.g., 200ms–1s).
2. **Read / socket timeout** — max idle time waiting for response bytes after connecting.
3. **Pool acquisition timeout** — max time waiting to borrow a connection from an HTTP or DB pool (prevents invisible queueing).
4. **End-to-end request deadline** — total wall-clock budget for the entire operation, including all [[Retry Pattern|retries]] and downstream hops.

## Deadline Propagation

In a call chain `Gateway → Service A → Service B → DB`, static per-hop timeouts fail: if Gateway has a 2s timeout, A has 2s, and B has 2s, B can keep doing expensive work **after Gateway has already given up and returned 504 to the user** (wasted "zombie work").

Instead, propagate an **absolute deadline** (e.g., [[gRPC]] `grpc-timeout` header or HTTP deadline header) across hops so each downstream service sees the remaining budget and cancels work once the deadline expires.

> [!warning] Failure modes & trade-offs
> - **Too short**: Prematurely aborts healthy p99 requests, triggering retries that add load and create a **metastable retry storm** (especially dangerous when combined with non-idempotent writes whose server-side commit actually succeeded).
> - **Too long (or infinite default)**: Many HTTP clients and JDBC drivers ship with **no timeout** or huge defaults (60s+). One hung downstream silently drains every worker thread in seconds.
> - **Zombie work**: A client-side timeout only frees the caller; the server keeps executing unless cancellation/deadline propagation is wired through (e.g., SQL `statement_timeout`, gRPC cancellation).

## Why is deadline propagation better than independent static timeouts in a microservice call chain? #card
Static per-hop timeouts allow downstream services to keep executing expensive work after the upstream caller has already timed out ("zombie work"). Propagating the remaining **end-to-end deadline** (e.g., via [[gRPC]] headers) lets every hop budget its work and cancel immediately when the caller's deadline expires.

Setting a timeout too short can cause ==premature aborts and retry storms==, while omitting timeouts causes ==thread and connection pool exhaustion==.

Timeout is the ==first line of defense== in resilience: always set explicit connect, read, pool-wait, and end-to-end deadline budgets on every network and DB call. #card-reminder

## Related
- [[Microservices Resilience and Fault tolerance]]
- [[Circuit Breaker]]
- [[Retry Pattern]]
- [[Bulkhead Pattern]]
- [[Fallback Pattern]]
- [[gRPC]]
- [[Backpressure]]
