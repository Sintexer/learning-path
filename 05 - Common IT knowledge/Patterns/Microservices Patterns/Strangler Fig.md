---
flashcards:
  q-v7pw: { nid: 1795356721367, hash: egjn9tak, sync: ixd5a2b6 }
  q-krqh: { nid: 1795356721392, hash: nqbnu4jt, sync: ma2vx2nx }
---

> **Central question:** How do we replace a production system without rewriting and switching everything at once?

The **Strangler Fig pattern** gradually replaces an existing system by moving selected capabilities to a new implementation.

During migration, both systems coexist:

```
             Routing layer (API Gateway / Reverse Proxy)
                                  |
                     ┌────────────┴────────────┐
                     v                         v
               Legacy system              New services
            Remaining features         Migrated features
```

Over time, the new system takes over more responsibilities until the old system can be retired. The pattern does not require [[Microservices architecture|microservices]]. You can use it to replace a legacy application with a new modular monolith.

It might sound tempting to migrate all at once — develop a fully new service and cut over ("big-bang rewrite"). But this rarely succeeds. Many features are undocumented, hidden integrations exist, and edge cases accumulate and fail all at once. The Strangler Fig solves this by migrating one slice at a time, observing real production behavior, fixing gaps, and moving further only after a green signal.

Other patterns, like [[Shadow Testing]], [[Anti-Corruption Layer]], [[Change Data Capture]], and [[Reconciliation]] simplify the migration process.

> [!warning] Failure modes & trade-offs
> - **The "eternal coexistence" trap**: Stopping at 70% migrated leaves you paying the operational cost of **both** the legacy system and the new services indefinitely, plus maintaining sync pipelines between them. Define an explicit retirement milestone up front.
> - **Shared database entanglement**: Moving code without decoupling the underlying data ([[Database Per Service]]) leaves both systems writing to the same tables, causing schema-change gridlock and [[Dual-Write]] bugs. Use [[Change Data Capture]] + [[Anti-Corruption Layer]] to sync data cleanly during transition.

Strangler Fig Pattern:::Incrementally replacing a legacy system by placing a routing facade in front and migrating one capability at a time to new services while both coexist until the legacy system can be retired.
^q-v7pw

## Which supporting patterns make a Strangler Fig migration safe in production? #card
- **[[API Gateway]] / [[Reverse proxy]]**: Routes traffic incrementally between legacy and new slices.
- **[[Anti-Corruption Layer]]**: Prevents legacy schemas/semantics from leaking into the new domain model.
- **[[Shadow Testing]]**: Validates parity against live traffic before shifting real users.
- **[[Change Data Capture]] + [[Reconciliation]]**: Keeps data in sync during coexistence and catches drift.
^q-krqh

## Related
- [[Anti-Corruption Layer]]
- [[Shadow Testing]]
- [[Reconciliation]]
- [[API Gateway]]
- [[Database Per Service]]
- [[Change Data Capture]]
- [[Zero-downtime deployment]]
- [[Microservices decomposition]]
