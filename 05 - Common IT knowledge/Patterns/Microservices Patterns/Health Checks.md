**Purpose:** Tell infrastructure (e.g., [[Kubernetes]], load balancers) whether an instance should start receiving traffic, continue receiving traffic, or be restarted.

These are different questions:

| Probe | Question | Typical consequence of failure |
| --- | --- | --- |
| **Startup** | Has initialization completed? | Allow startup time; restart if it never succeeds |
| **Liveness** | Is this process stuck or irrecoverably unhealthy? | Restart the [[Pod]] / container |
| **Readiness** | Can this instance currently serve its intended traffic? | Stop routing traffic to it (remove from [[K8S Service]] endpoints) |

Spring Boot Actuator provides health endpoints and supports liveness/readiness groups, commonly exposed as:

```http
/actuator/health/liveness
/actuator/health/readiness
```

### Important limitation: do not create restart cascades

If Payment Service is down, restarting every Order Service instance usually achieves nothing — it only makes the outage worse by wiping caches and triggering thundering herds on boot.

Therefore:
- **Liveness** should generally **not** depend on remote services (check only internal deadlocks, corrupted in-process state, or unrecoverable event-loop stalls).
- **Readiness** should include dependencies only when their failure truly prevents useful service (and even then, be careful: if a shared DB blips for 2 seconds and every pod marks itself unready, traffic drops to 0% and reconnect storms follow).

For example, Order Service may still accept a pending order while payment is unavailable.

This connects to **[[Circuit Breaker|Circuit Breakers]]** and **graceful degradation** ([[Fallback Pattern]]).

Also, a successful health check does not prove business correctness. A healthy process can still have a growing [[Outbox Pattern|outbox]] backlog or stuck [[Saga|Sagas]]; those require metrics, [[Distributed Tracing]], and alerts.

> [!warning] Failure modes & trade-offs
> Putting a downstream DB or HTTP dependency inside a **liveness** probe is one of the most common Kubernetes self-inflicted outages: a 5-second dependency blip causes kubelet to kill and restart the entire fleet simultaneously, turning a minor blip into a minutes-long cascading outage.

## Why should a Kubernetes Liveness probe almost never check downstream databases or external APIs? #card
A failed liveness probe triggers a **container restart**. If a downstream DB or API blips, every pod fails liveness and restarts simultaneously — creating a fleet-wide restart storm and cold-start thundering herd that cannot fix the remote dependency.

Liveness answers =="is this process irrecoverably stuck?"== (restart it), whereas Readiness answers =="can this instance serve traffic right now?"== (stop routing to it).

## Related
- [[Kubernetes]]
- [[Pod]]
- [[K8S Service]]
- [[Load Balancing]]
- [[Circuit Breaker]]
- [[Fallback Pattern]]
- [[Zero-downtime deployment]]
- [[Microservices Resilience and Fault tolerance]]
