**Purpose:** Combine information from multiple sources into one response.

```
Order Overview Aggregator
    ├── Order Service
    ├── Payment Service
    └── Shipping Service
```

It reduces client-side complexity and network round trips.

A [[BFF]] often acts as an aggregator, but aggregation can also serve internal APIs (e.g., behind an [[API Gateway]]).

### Practical limitations

- **Fan-out load amplification**: 1 incoming request becomes \(N\) downstream requests.
- **Tail-latency amplification**: Required dependencies directly govern latency and [[Availability]] — the response waits for the slowest required call (and overall availability is the product of required services' availabilities).
- **No consistent snapshot**: Concurrent calls observe each service at slightly different timestamps; responses do not form a globally consistent snapshot.
- **Partial failures**: Must be explicitly represented (e.g., omit non-critical widgets via [[Fallback Pattern]] rather than failing the whole response).

Use end-to-end deadlines ([[Timeout Pattern]]), bounded concurrency ([[Bulkhead Pattern]]), [[Circuit Breaker|circuit breakers]], and truthful fallbacks.

If the same expensive aggregation happens frequently, consider a **materialized [[CQRS]] projection** instead.

> [!warning] Failure modes & trade-offs
> If an Aggregator calls 5 required services in parallel, each with a 99% p99 SLA or 99.9% availability, the composite p99 tail latency and failure probability multiply rapidly. Classify every downstream call as **required** vs **optional** up front and degrade optional branches cleanly.

## What are the two biggest runtime risks of a live Aggregator, and when should you replace it with CQRS? #card
1. **Fan-out load & tail-latency amplification** (slowest required call dictates latency; 1 request multiplies into \(N\) downstream calls).
2. **Compound unavailability** (failing any required dependency fails the whole request).
Replace live aggregation with a **materialized [[CQRS]] projection** when the composed view is read-heavy, expensive to assemble, and can tolerate [[Eventual Consistency]].

A live Aggregator calling multiple services concurrently does ==not== produce a globally consistent snapshot; responses reflect each service at slightly different points in time. #card-reminder

## Related
- [[BFF]]
- [[API Gateway]]
- [[CQRS]]
- [[Database Per Service]]
- [[Timeout Pattern]]
- [[Bulkhead Pattern]]
- [[Fallback Pattern]]
- [[Availability]]
