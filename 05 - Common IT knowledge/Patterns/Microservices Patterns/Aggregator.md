**Purpose:** Combine information from multiple sources into one response.

```
Order Overview Aggregator
    ├── Order Service
    ├── Payment Service
    └── Shipping Service
```

It reduces client-side complexity and network round trips.

A [[BFF]] often acts as an aggregator, but aggregation can also serve internal APIs.

### Practical limitations

- Fan-out increases downstream load.
- Required dependencies affect latency and availability.
- Responses do not form a globally consistent snapshot.
- Partial failures need explicit representation.

Use deadlines, bounded concurrency, and truthful fallbacks.

If the same expensive aggregation happens frequently, consider a **materialized [[CQRS]] projection** instead.
