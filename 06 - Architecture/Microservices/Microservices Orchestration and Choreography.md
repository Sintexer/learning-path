---
flashcards:
  q-rawa: { nid: 1795356717791, hash: 689ewd6m, sync: s9bpncfi }
  q-p675: { nid: 1795356717816, hash: i4vx86az, sync: w8hid7kp }
---

Service **orchestration** and **choreography** are two approaches to managing multi-step interactions and [[Saga|Sagas]] across [[Microservices architecture|microservices]].

## Service Orchestration

- **Centralized Control**: A central service (orchestrator) explicitly commands each participating service and tracks workflow state (e.g., in a [[Saga State Table]]).
- **Workflow Management**: The orchestrator defines the happy path, retries, timeouts, and compensation sequence in one place — making complex business processes far easier to inspect, debug, and change.
- **Examples**: [[Temporal]], Camunda (BPMN), AWS Step Functions, or a custom Spring state machine backed by [[Outbox Pattern]].

## Service Choreography

- **Decentralized Control**: No central coordinator. Each service listens for domain events, performs its local action, and publishes its own events for downstream services to react to ([[Event-driven architecture]]).
- **Event-Driven**: Services publish and subscribe via a broker like [[Kafka]] or RabbitMQ, yielding lower upfront coupling for simple 2–3 step reactions.

## Comparison

| Dimension | Orchestration | Choreography |
| --- | --- | --- |
| **Control flow** | Explicit, centralized in one codebase/state machine | Implicit, spread across multiple services' event handlers |
| **Observability** | High — query one workflow state to see where an order is | Harder — requires [[Distributed Tracing]] across many topics |
| **Coupling** | Orchestrator knows participants (commands) | Participants stay decoupled, but share event schema contracts |
| **Adding/reordering steps** | Change one orchestrator | Coordinate deployments across multiple services |
| **Best fit** | Complex multi-step workflows, payments, explicit [[Saga]] compensations | Simple broadcast reactions (analytics, notifications, [[CQRS]] projections) |

> [!warning] Failure modes & trade-offs
> - **Choreography "pinball architecture"**: Once a choreographed workflow grows beyond ~3–4 steps with conditional branches and compensations, no single engineer can tell the full execution path, and cyclic event loops become easy to introduce accidentally.
> - **Orchestration "God service"**: Cramming domain business logic (pricing, inventory rules, fraud math) into the orchestrator turns it into a monolith. Keep the orchestrator thin: it owns **workflow progression and failure handling**, while domain services own **domain invariants**.
> - **Hybrid in practice**: Most mature systems **orchestrate** within a core transactional workflow (e.g., checkout [[Saga]]) and **choreograph** side effects across bounded contexts (e.g., emit `OrderCompleted` for analytics, email, and [[CQRS]] read models).

## When should you choose Orchestration over Choreography for a multi-service workflow? #card
Choose **Orchestration** when a workflow has 4+ steps, conditional branching, strict step ordering, or explicit [[Saga]] compensation — because keeping state transitions in one place (e.g., [[Temporal]] or a [[Saga State Table]]) prevents "pinball architecture" and makes failures easy to inspect and recover. Use **Choreography** for loosely coupled fan-out reactions (notifications, analytics, [[CQRS]] projections).
^q-rawa

Keep an orchestrator focused on ==workflow sequencing and failure recovery== — never let it absorb ==domain business rules== that belong in the participating services. #card-reminder
^q-p675

## Related
- [[Saga]]
- [[Temporal]]
- [[Saga State Table]]
- [[Event-driven architecture]]
- [[Outbox Pattern]]
- [[Kafka]]
- [[Distributed Tracing]]
- [[Microservices patterns]]
