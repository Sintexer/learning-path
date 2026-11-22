**The idea:** When a downstream call fails, times out ([[Timeout Pattern]]), or the [[Circuit Breaker]] is open, don't just propagate a raw error — degrade gracefully instead.

**Have this example ready, verbatim style:**

> "If the live odds service is down, fall back to the last cached odds with a 'may be outdated' flag, rather than showing an error to the user."

That's a strong answer because it's concrete and shows product thinking, not just technical mechanism.

## Common Fallback Strategies

1. **Stale cache / last-known-good** — serve slightly stale data from [[Redis]] or local memory with an explicit staleness indicator (best for read-heavy, non-financial views).
2. **Static / default response** — e.g., generic "Popular items" instead of personalized recommendations when the ML service is down.
3. **Graceful degradation (feature shedding)** — omit non-critical widgets in an [[Aggregator]] or [[BFF]] response while keeping the core page functional.
4. **Deferred / async queueing** — accept the request into a durable queue ([[Kafka]]) for later processing and return `202 Accepted` instead of failing synchronously.
5. **Fail fast (no fallback)** — for safety-critical writes (payments, auth, balance checks), returning fabricated or stale data is worse than failing cleanly.

> [!warning] Failure modes & trade-offs
> - **Untested fallback paths**: A fallback that only runs during a real outage often fails itself (stale schema, missing permissions, unindexed query) or overloads a backup dependency under 100% failover traffic. Test fallbacks via chaos / fault injection.
> - **Dangerous silent degradation**: Never silently fall back on writes, pricing/quoting, or authorization — stale or permissive defaults can cause financial loss or security breaches.
> - **Hidden outages**: If fallbacks swallow errors without incrementing dedicated metrics/alerts, the system can run degraded for days before anyone notices.

## When should you deliberately NOT use a Fallback and instead fail fast? #card
For safety-critical or transactional operations where stale or default values cause worse harm than an error — such as **payments, authorization/authentication checks, inventory reservations, and live financial execution**.

Every fallback path must emit ==metrics/alerts== and be ==tested under load==, otherwise it will silently mask outages or fail when an incident actually hits. #card-reminder

## Related
- [[Microservices Resilience and Fault tolerance]]
- [[Circuit Breaker]]
- [[Timeout Pattern]]
- [[Bulkhead Pattern]]
- [[Aggregator]]
- [[BFF]]
- [[Availability]]
