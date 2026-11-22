---
flashcards:
  q-qysp: { nid: 1795356717667, hash: 9mefsjay, sync: cvtunmdt }
  q-pe8q: { nid: 1795356717692, hash: yc6exps3, sync: ynjsqxdi }
---

Resilience and fault tolerance are critical aspects of [[Microservices architecture]], ensuring that the system handles partial failures gracefully instead of cascading into a platform-wide outage.

## Core Resilience Patterns

- **[[Timeout Pattern|Timeouts]]**: First line of defense — bound every connect, read, pool-wait, and end-to-end deadline so hung dependencies cannot exhaust threads.
- **[[Circuit Breaker]]**: Stops calling a failing or slow dependency so callers fail fast and avoid cascading failures.
- **[[Retry Pattern|Retries]]**: Recover from transient blips using **exponential backoff + jitter**, only for [[Idempotency|idempotent]] operations ([[Idempotency Key]]), and only at one layer with a retry budget.
- **[[Bulkhead Pattern|Bulkheads]]**: Isolate thread pools, semaphores, or connection pools per dependency/tenant so one slow downstream cannot starve unrelated traffic.
- **[[Fallback Pattern|Fallbacks]]**: Degrade gracefully (stale cache, feature shedding) for non-critical reads, while failing fast on safety-critical writes.
- **[[Rate Limiting]] & [[Backpressure]]**: Protect services from overload by shedding excess traffic at the edge ([[API Gateway]]) or signaling producers to slow down.
- **[[Health Checks]]**: Separate startup, liveness, and readiness probes without triggering restart cascades on downstream blips.

## Pattern Composition Order (Resilience4j)

Order matters when wrapping a remote call! A standard outer-to-inner execution stack is:

```text
Fallback (outermost — catches final failure)
  └── Retry (retries transient failures / timeouts)
        └── Circuit Breaker (records failures; short-circuits when Open)
              └── Rate Limiter / Bulkhead (caps concurrency)
                    └── TimeLimiter / Timeout (innermost — bounds each attempt)
                          └── Remote Call
```

> [!warning] Failure modes & trade-offs
> Putting **Circuit Breaker outside Retry** vs **inside Retry** changes behavior:
> - **Retry outside Circuit Breaker**: Each failed attempt counts toward the breaker's sliding window; once the breaker opens, subsequent retries immediately see `CallNotPermittedException` (stop retrying if you exclude that exception from retry).
> - **Double resilience (App + [[Service Mesh]])**: Configuring retries in *both* Resilience4j and Envoy/Istio silently multiplies retries (\(3 \times 3 = 9\)). Pick **one** layer to own retries and timeouts.

## Tools and Libraries

- **Resilience4j**: Lightweight, modular fault-tolerance library for Java/Kotlin/Spring (standard modern choice).
- **Hystrix**: Older Netflix library (maintenance mode; historical reference only).
- **[[Service Mesh]] (Envoy / Istio / Linkerd)**: Infrastructure-level timeouts, retries, and outlier detection (see [[Microservices Service mesh pattern]]).

## In what order should Resilience4j decorators typically wrap a remote call (outermost to innermost)? #card
**Fallback → Retry → Circuit Breaker → Bulkhead / Rate Limiter → Timeout → Remote Call**.
Timeout bounds each individual attempt; Bulkhead caps concurrency; Circuit Breaker tracks failures and short-circuits; Retry re-attempts transient failures (while skipping `CallNotPermittedException`); Fallback handles the final failure gracefully.
^q-qysp

Never configure retries in ==both application code and the Service Mesh== without coordination, or retries multiply silently into retry storms. #card-reminder
^q-pe8q

## Related
- [[Timeout Pattern]]
- [[Circuit Breaker]]
- [[Retry Pattern]]
- [[Bulkhead Pattern]]
- [[Fallback Pattern]]
- [[Health Checks]]
- [[Rate Limiting]]
- [[Backpressure]]
- [[Idempotency Key]]
- [[Microservices Service mesh pattern]]
- [[Availability]]
