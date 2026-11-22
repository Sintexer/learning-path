---
flashcards:
  q-x9ew: { nid: 1795356721492, hash: bwy5qea2, sync: mgskucji }
  q-87ig: { nid: 1795356721518, hash: secanij9, sync: uhauqh3q }
---

Each service owns its persistent data and controls access to it (a foundational rule of [[Microservices architecture]] and [[DDD]] bounded contexts).

```
Order Service   → Order database
Payment Service → Payment database
```

Other services use APIs or events — not direct table access.

### Does this require separate database servers?

No. Services can share a database cluster while using separate databases or schemas and dedicated credentials ([[Least privilege principle]]).

The important distinction is **exclusive ownership**, although shared infrastructure still creates resource contention (noisy neighbors) and a shared failure domain (mitigated by [[Bulkhead Pattern]] / separate instances for critical tiers).

### Why use it?

- Independent [[Schema Migration|schema changes]] and deployments.
- Clear responsibility for business invariants.
- Freedom to choose suitable storage (polyglot persistence — e.g., [[PostgreSQL]] for orders, [[Redis]] for sessions, [[MongoDB]] for catalogs).
- Prevention of hidden coupling through shared SQL tables.

### Practical limitations

You lose convenient cross-service:
- Joins.
- Foreign keys.
- Local [[ACID]] [[Transaction|transactions]].

Replace them selectively with:
- **[[Aggregator|API aggregation]]** (or [[BFF]]) for composed reads.
- **[[CQRS]] projections** for query-heavy cross-service views.
- **[[Saga|Sagas]]** (see [[2PC vs Saga]]) for multi-service workflows.
- **[[Outbox Pattern|Outbox events]]** / [[Change Data Capture]] for reliable propagation without [[Dual-Write]].

> [!warning] Failure modes & trade-offs
> **Shared-database backdoor**: The moment Service B queries Service A's table directly "just for reporting," Service A loses the ability to refactor its schema safely. Enforce ownership with separate DB users/roles and expose read needs via APIs, events, or dedicated [[Data Warehouse]] / [[ETL]] pipelines.

Database Per Service:::A microservice pattern where each service has exclusive ownership of its schema/data store and other services must interact only through its APIs or events, never direct SQL access.
^q-x9ew

## What three database conveniences do you give up with Database Per Service, and what replaces them? #card
You give up cross-service **SQL joins**, **foreign keys**, and **local [[ACID]] transactions**.
Replace them with:
- **[[Aggregator]] / [[BFF]]** or **[[CQRS]] projections** (for joins/reads)
- **[[Saga]]** (for multi-service transactions)
- **[[Outbox Pattern]] / [[Change Data Capture]]** + **[[Reconciliation]]** (for cross-service consistency)
^q-87ig

## Related
- [[Microservices architecture]]
- [[Microservices decomposition]]
- [[DDD]]
- [[CQRS]]
- [[Saga]]
- [[2PC vs Saga]]
- [[Outbox Pattern]]
- [[Schema Migration]]
- [[Aggregator]]
