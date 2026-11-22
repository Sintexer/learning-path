---
flashcards:
  q-km53: { nid: 1795356722068, hash: n2df8fej, sync: tdj2up8e }
  q-v9ek: { nid: 1795356722093, hash: ewtpr9eh, sync: dr2dykrc }
  q-3dsy: { nid: 1795356722119, hash: 9ufra9xz, sync: ada4zk56 }
---

A **BFF (Backend for Frontend)** adapts backend capabilities to the needs of a particular frontend.

Why separate them? Mobile and web clients may differ in:
- Screen layouts.
- Network constraints.
- Payload size requirements.
- Feature-release schedules.
- [[Backward Compatibility|Backward compatibility]] needs.

A mobile application might remain installed for months without updating. A web frontend can often be deployed centrally.

A dedicated BFF can absorb those differences without forcing every domain service to expose client-specific endpoints (unlike a general-purpose [[API Gateway]]).

## Aggregation

Basically, any BFF acts as an [[Aggregator]]. An order page might require:

```text
Order details
Payment summary
Shipment estimate
```

A BFF can expose:

```http
GET /mobile/orders/O123
```

Internally:

```text
Load order
    |
    ├── Load authorized payment summary
    └── Load shipment estimate
    |
Combine into mobile response
```

Independent calls can run concurrently to reduce latency.

But parallel fan-out creates an important dependency:

> The response is often constrained by the slowest required call.

It also creates **load amplification**:

```text
1 client request
    → 5 downstream requests
```

At 1,000 client requests per second, that endpoint may generate 5,000 downstream requests per second.

This connects directly to [[Microservices Resilience and Fault tolerance]]:
- Set an overall request deadline ([[Timeout Pattern]]).
- Give downstream calls bounded timeouts.
- Apply [[Bulkhead Pattern|bulkheads]] to expensive dependencies.
- Use [[Circuit Breaker|Circuit Breakers]] where appropriate.
- Avoid [[Retry Pattern|retries]] at every layer.

A reactive client such as Spring `WebClient` (or Java [[Virtual Threads]]) can support concurrent nonblocking requests, but nonblocking I/O does not eliminate downstream capacity limits.

> [!warning] Failure modes & trade-offs
> **BFF creep**: When teams put core business rules (authorization logic, pricing calculations, state transitions) inside a BFF, those rules get duplicated across Web BFF and Mobile BFF and drift out of sync. Keep BFFs strictly for **orchestration, response shaping, and client-specific adaptation** — never as the source of domain truth.

## Relation to CQRS

BFF aggregation does not produce a consistent snapshot. Suppose the BFF reads:

```text
Order Service:    order is CONFIRMED
Payment Service: payment is REFUNDED
```

Those responses may have been captured at different moments during a [[Saga]]. Calling services concurrently does not create a distributed read transaction. The BFF must tolerate legitimate intermediate states.

Instead of querying five services for every request, use [[CQRS]]:

```text
Domain events
    → Order overview projection
    → BFF reads one view
```

This can reduce fan-out and improve predictable latency. But the projection is [[Eventual Consistency|eventually consistent]]. The trade-off is:

| Live aggregation | Materialized projection |
| --- | --- |
| Calls source APIs at request time | Reads a precomputed view |
| Can reflect recent source state | May lag behind source services |
| Dependency failures affect requests directly | Can serve during some source outages |
| Higher fan-out cost | More event-processing and [[Reconciliation\|reconciliation]] work |

A projection is not automatically a globally consistent snapshot either. Its consistency depends on the event model and how updates are applied.

Backend for Frontend (BFF):::A dedicated backend service tailored to a single client type (e.g., mobile vs web) that aggregates and shapes downstream APIs without polluting core domain services with UI-specific endpoints.
^q-km53

## Live BFF Aggregation vs. Materialized CQRS Projection — what is the core trade-off? #card
- **Live aggregation**: Fresher data and simpler infrastructure, but suffers from **fan-out load amplification**, tail-latency compounding, and direct exposure to downstream outages.
- **Materialized [[CQRS]] projection**: Predictable low latency and resilience to source outages, but introduces **[[Eventual Consistency]]** (read lag) and projection/[[Reconciliation]] operational overhead.
^q-v9ek

Keep BFFs limited to ==client-specific aggregation and response shaping==; never duplicate core domain rules across multiple BFFs. #card-reminder
^q-3dsy

## Related
- [[API Gateway]]
- [[Aggregator]]
- [[CQRS]]
- [[GraphQL]]
- [[Microservices Resilience and Fault tolerance]]
- [[Saga]]
- [[Eventual Consistency]]
