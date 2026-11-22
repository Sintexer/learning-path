---
flashcards:
  q-akck: { nid: 1795356721617, hash: upejjkgv, sync: usvgh8rs }
  q-m2rb: { nid: 1795356721642, hash: gqzhyujq, sync: py8mnb4s }
---

**Purpose:** Reuse code across services without adding a network dependency.

Good candidates include:
- Logging and [[Distributed Tracing|tracing]] setup (e.g., [[OpenTelemetry]] conventions).
- Small security integration helpers (e.g., [[JWT]] verification).
- Standard client instrumentation (metrics, [[Timeout Pattern]] / [[Circuit Breaker]] defaults).
- Carefully scoped technical utilities.

### Why use it?

A library offers local execution (zero network hop) and consistent implementation of repeated technical concerns.

### Practical limitations

A library duplicates its code into every consuming deployment.

Therefore:
- Fixes require upgrading and redeploying **every** consumer (unlike updating a network service or [[Microservices Sidecar pattern|sidecar]]).
- Different services may run different versions simultaneously (version skew).
- Large shared libraries can force coordinated releases — recreating a **distributed monolith**.

Be particularly cautious with shared:
- ORM entities (e.g., [[JPA]] / [[Hibernate]] models).
- Repository layers.
- Domain models.
- Business rules owned by another service.

For example, sharing Payment Service's internal entity classes couples consumers to its persistence design (violating [[Database Per Service]]).

Prefer explicit [[API]] / event contracts, with [[Backward Compatibility|backward-compatible]] evolution ([[Protocol Buffers]] / OpenAPI) and contract tests.

> [!warning] Failure modes & trade-offs
> **Transitive dependency hell**: A "common-utils" library that pulls in heavy frameworks (Spring, Jackson, Hibernate, AWS SDK) creates classpath conflicts and locks every service to the same framework upgrade cycle. Keep shared libraries tiny, focused, and minimal in dependencies — or move cross-cutting network concerns to a [[Microservices Sidecar pattern|sidecar]] / [[Service Mesh]].

## What belongs in a Shared Library across microservices vs. what should NEVER be shared? #card
- **Good candidates**: Thin, stable technical utilities (logging/[[OpenTelemetry]] setup, [[JWT]] parsing, HTTP client instrumentation).
- **Never share**: Internal ORM/[[JPA]] entities, repositories, or domain business rules — these create tight coupling, break [[Database Per Service]], and force coordinated deployments across teams.
^q-akck

Remember: a Shared Library gives ==zero network overhead== at the cost of ==redeploying every consumer== whenever a fix is released. #card-reminder
^q-m2rb

## Related
- [[Microservices architecture]]
- [[Database Per Service]]
- [[Microservices Sidecar pattern]]
- [[Backward Compatibility]]
- [[Composition Over Inheritance]]
- [[API Gateway]]
