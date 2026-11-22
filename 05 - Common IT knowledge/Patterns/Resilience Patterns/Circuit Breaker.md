---
flashcards:
  q-6bdg: { nid: 1795356721117, hash: yyv7yh2v, sync: 9py79eyk }
  q-ifsr: { nid: 1795356721142, hash: pcca33d7, sync: b7w4wbhs }
  q-bcng: { nid: 1795356721168, hash: nzzu6274, sync: rwzhgq5p }
---

> Prevents cascading failures

**The problem it solves:** Imagine Service A calls Service B, and B is dying — every call to B takes 30 seconds to time out. If A keeps calling B anyway, A's threads pile up waiting on B, A's thread pool exhausts, and now A itself becomes unresponsive to _its_ callers. One failing service cascades upward and takes down the whole chain.

**The fix:** Stop calling the broken thing. Fail fast instead (and optionally invoke a [[Fallback Pattern|fallback]]).

**Three states:**

| State         | Behavior                                                                                                        |
| ------------- | --------------------------------------------------------------------------------------------------------------- |
| **Closed**    | Normal — calls go through, failures and slow calls are counted in a sliding window                              |
| **Open**      | Threshold exceeded — calls short-circuit immediately, no attempt made, usually returns fallback/error instantly |
| **Half-Open** | After a cooldown, a few test calls are let through. Success -> back to Closed. Failure -> back to Open.         |

Think of it like an electrical circuit breaker in your house — trips when there's too much current (too many failures), and you have to manually (or here, automatically after a timeout) test if it's safe to reset.

**Libraries:** Resilience4j (the modern standard in the Java/Kotlin/Spring world — Netflix Hystrix is the older one, now in maintenance mode). If someone asks "how would you implement this in Spring," that's your answer. In Kubernetes, [[Service Mesh|service meshes]] (Envoy/Istio) also provide outlier detection at the proxy layer (see [[Microservices Service mesh pattern]]).

> [!warning] Failure modes & trade-offs
> - **Half-Open thundering herd**: If every instance of Service A opens at the same time and shares the same cooldown, all instances enter Half-Open simultaneously and flood recovering Service B with probe traffic. Add jitter to the wait duration and limit `permittedNumberOfCallsInHalfOpenState`.
> - **Wrong failure classification**: Counting `4xx` business validation errors as circuit failures trips the breaker during bad client input instead of real outages. Only trip on `5xx`, timeouts, and network errors — and configure a **slow-call rate threshold** so brownouts (high latency, 200 OK) also trip it.
> - **Per-instance vs fleet view**: Local in-memory breakers don't share state across pods; with low per-pod traffic, set `minimumNumberOfCalls` carefully so a single blip doesn't trip the circuit.

Circuit Breaker states:::Closed (normal, tracking failure/slow-call rate), Open (fail fast, short-circuit calls), and Half-Open (allow limited probe calls after cooldown to test recovery).
^q-6bdg

## Why can a Circuit Breaker's Half-Open state re-crash a recovering downstream service? #card
If many caller instances trip simultaneously and use the same fixed cooldown, they all transition to **Half-Open** at the exact same moment and send a burst of probe requests (a **thundering herd**) that overwhelms the barely recovered dependency. Mitigate with cooldown jitter, a small half-open probe limit, and slow-start warm-up.
^q-ifsr

A Circuit Breaker must count ==5xx, timeouts, and slow calls== as failures, never ==4xx client validation errors==, or bad user input will trip the breaker for everyone. #card-reminder
^q-bcng

## Related
- [[Microservices Resilience and Fault tolerance]]
- [[Timeout Pattern]]
- [[Retry Pattern]]
- [[Bulkhead Pattern]]
- [[Fallback Pattern]]
- [[Health Checks]]
- [[Microservices Service mesh pattern]]
- [[Service Mesh]]
