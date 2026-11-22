---
flashcards:
  q-pmds: { nid: 1795356720993, hash: r4gay2zv, sync: z9jqjpzi }
  q-4xna: { nid: 1795356721017, hash: 3h8kut4g, sync: 94q85ccn }
  q-ctn6: { nid: 1795356721043, hash: ycsjxghm, sync: nuqcachr }
---

Named after ship compartments — if one compartment floods, bulkheads (physical walls) stop the whole ship from sinking.

**Applied to software:** give each downstream dependency (or tenant/workload) **its own isolated pool** of threads, connections, or concurrency permits. If Service B is slow and eating up all its allocated threads, that doesn't touch the threads reserved for calling Service C — unrelated requests stay healthy.

## Two Implementation Styles

| Style | Mechanism | Isolation Level | Trade-offs |
| --- | --- | --- | --- |
| **Thread-pool bulkhead** | Dedicated executor + bounded queue per dependency | Strong — enforces timeouts and prevents caller threads from blocking indefinitely | Extra context-switching & memory overhead per pool |
| **Semaphore bulkhead** | Bounded concurrent permit counter on the caller's thread (e.g., Resilience4j `SemaphoreBulkhead`) | Lightweight — caps concurrent in-flight calls | Does **not** interrupt a hung call; relies on strict [[Timeout Pattern\|timeouts]] |

At infrastructure level, bulkheads also apply to **separate DB connection pools** (e.g., read vs write, or critical checkout vs reporting queries), **separate [[Kafka Consumer|consumer]] groups/pods** for noisy tenants, and **cell-based / zonal isolation**.

## Sizing & Composition

- Size pools using [[Little's Law]] (\(L = \lambda \cdot W\)): concurrency limit ≈ peak target RPS × p99 latency + small headroom. Keep queues **tiny or zero** — a deep queue hides overload and turns fast failures into latency spikes.
- Combine with [[Timeout Pattern]] (so occupied slots are freed), [[Circuit Breaker]] (so callers stop hammering a broken dependency), [[Backpressure]] / [[Rate Limiting]], and [[Fallback Pattern]].

> [!warning] Failure modes & trade-offs
> **Over-fragmentation** wastes idle threads/connections across dozens of micro-pools, while **shared hidden bottlenecks** (e.g., all bulkheads sharing the same underlying DB connection pool, DNS resolver, or HTTP client pool) silently defeat the isolation. Always trace resource pools end-to-end.

Bulkhead Pattern:::Isolating resources (thread pools, semaphores, connection pools, or pods) per dependency or tenant so a slow or failing component cannot exhaust shared capacity and take down unrelated traffic.
^q-pmds

## When should you choose a Semaphore Bulkhead over a Thread-Pool Bulkhead? #card
Choose a **Semaphore Bulkhead** for lightweight concurrency capping when calls are non-blocking or strictly bounded by fast [[Timeout Pattern|timeouts]] (e.g., reactive/[[Virtual Threads]] stacks), avoiding context-switch overhead. Choose a **Thread-Pool Bulkhead** when calling blocking/untrusted I/O where you need hard thread isolation and caller-side cancellation.
^q-4xna

Never put an unbounded queue in front of a bulkhead; deep queues hide overload and convert ==fast rejections== into ==tail-latency spikes==. #card-reminder
^q-ctn6

## Related
- [[Microservices Resilience and Fault tolerance]]
- [[Circuit Breaker]]
- [[Timeout Pattern]]
- [[Fallback Pattern]]
- [[Retry Pattern]]
- [[Little's Law]]
- [[Backpressure]]
- [[Rate Limiting]]
