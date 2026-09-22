A BFF adapts backend capabilities to the needs of a particular frontend.

Why separate them? Mobile and web clients may differ in:
- Screen layouts.
- Network constraints.
- Payload size requirements.
- Feature-release schedules.
- Backward compatibility needs.

A mobile application might remain installed for months without updating. A web frontend can often be deployed centrally.

A dedicated BFF can absorb those differences without forcing every domain service to expose client-specific endpoints.

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

This connects directly to the previous chapter:

- Set an overall request deadline.
- Give downstream calls bounded timeouts.
- Apply bulkheads to expensive dependencies.
- Use Circuit Breakers where appropriate.
- Avoid retries at every layer.

A reactive client such as Spring `WebClient` can support concurrent nonblocking requests, but nonblocking I/O does not eliminate downstream capacity limits.

## Relation to CQRS

BFF Aggregation Does Not Produce a Consistent Snapshot. Suppose the BFF reads:

```text
Order Service:    order is CONFIRMED
Payment Service: payment is REFUNDED
```

Those responses may have been captured at different moments during a [[Saga pattern|Saga]]. Calling services concurrently does not create a distributed read transaction. The BFF must tolerate legitimate intermediate states.

Instead of querying five services for every request, use [[CQRS]]:

```text
Domain events
    → Order overview projection
    → BFF reads one view
```

This can reduce fan-out and improve predictable latency. But the projection is eventually consistent. The trade-off is:

| Live aggregation | Materialized projection |
|---|---|
| Calls source APIs at request time | Reads a precomputed view |
| Can reflect recent source state | May lag behind source services |
| Dependency failures affect requests directly | Can serve during some source outages |
| Higher fan-out cost | More event-processing and reconciliation work |

A projection is not automatically a globally consistent snapshot either. Its consistency depends on the event model and how updates are applied.
