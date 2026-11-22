**Service decomposition** is the process of breaking down a monolithic application (or designing a new domain) into cohesive, loosely coupled [[Microservices architecture|microservices]]. Each microservice should encapsulate a clear business capability and own its data ([[Database Per Service]]).

## Approaches to Decomposition

- **By Business Capability**: Align services with stable business functions (e.g., Order Management, Billing, Inventory, Shipping) rather than technical layers (UI / Business / DB).
- **By [[DDD]] Subdomain / Bounded Context**: Use Strategic [[DDD]] to identify Core, Supporting, and Generic subdomains and draw service boundaries around **Bounded Contexts** where models and Ubiquitous Language diverge.
- **By Rate of Change (Volatility) & Team Ownership**: Apply Conway's Law intentionally — group code that changes together for the same business reason under one team so features don't require lockstep deployments across 5 teams.
- **By Non-Functional Requirements (Scaling, Security, Compliance)**: Isolate hot-path or sensitive workloads (e.g., PCI-scoped Payments, high-throughput live odds feed, ML inference) when their scaling profile or blast-radius requirements differ drastically.

## Decomposition Anti-Patterns (Warning Signs of Bad Boundaries)

1. **Entity / Table-per-Service ("Nano-services")**: Creating a `UserService`, `AddressService`, and `OrderLineService` forces constant chatty synchronous joins and distributed transactions for simple operations.
2. **Chatty Synchronous Call Chains**: One user request bouncing synchronously through 6+ services multiplies tail latency and couples [[Availability]] (e.g., \(0.999^6 \approx 0.994\)).
3. **Lockstep Deployments**: If changing one business feature always requires deploying Service A, B, and C together, they are **one logical service** split along the wrong seam.
4. **Shared Mutable Database**: Two services reading and writing the same tables destroys schema autonomy.

## Safe Migration Strategy

Never try to decompose a legacy monolith in one big-bang rewrite. First establish clean internal module boundaries (a **modular monolith**), then extract one high-value seam at a time using [[Strangler Fig]], [[Anti-Corruption Layer]], [[Change Data Capture]], and [[Shadow Testing]].

> [!warning] Failure modes & trade-offs
> **Premature decomposition** is far more expensive to undo than keeping modules together in a monolith: merging two services back together is painful, whereas splitting a well-modularized monolith once domain boundaries stabilize is straightforward. When in doubt, err slightly **coarser** (fewer, deeper services) rather than finer.

## What are three concrete signals that your microservice decomposition boundaries are drawn in the wrong place? #card
1. **Lockstep deployments**: One feature routinely requires coordinated releases across multiple services.
2. **Chatty synchronous chains**: Simple reads/writes require multi-hop RPCs or cross-service joins (classic entity/table-per-service mistake).
3. **Shared mutable tables**: Multiple services directly read/write the same database schema, breaking [[Database Per Service]].

Align service boundaries with ==DDD Bounded Contexts and transactional invariants== rather than ==individual database tables or technical layers==. #card-reminder

## Related
- [[Microservices architecture]]
- [[DDD]]
- [[Database Per Service]]
- [[Strangler Fig]]
- [[Anti-Corruption Layer]]
- [[2PC vs Saga]]
- [[Microservices patterns]]
