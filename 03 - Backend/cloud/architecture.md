# Cloud architecture

## Designing [[Microservices architecture|microservice architecture]]

1. **[[Microservices decomposition|Decomposing business problem]].** O -> o,o,o,o
2. **Establishing service granularity.** o o o o
3. **Defining the service interfaces.** Web | [[API]] | [[API]]

### Decomposing business problem

> *Decomposing:* 
> O -> o,o,o,o

Break big problem into several small ones — break monolithic domain into MS domain (guided by [[DDD]] bounded contexts).

The nouns you use become domain models.

The verbs you use become MS operations.

Look for data relations (strong vs. weak coupling).

### Service granularity

Shouldn't take too much responsibility. Shouldn't be too fine-grained (anemic CRUD services).

Rule of thumb: 1 service — 1 DB ([[Database Per Service]]) and cohesive tables owned exclusively by that service.

**Focus on service-to-service interactions** (minimize chatty synchronous fan-out).

### Defining the service interface

- Embrace [[REST]] / [[gRPC]] depending on public vs. internal latency needs.
- Use [[URI|URIs]] to address resources cleanly.
- Use JSON (lightweight, human-readable) or [[Protocol Buffers]] (compact, strongly typed).
- Use proper [[HTTP]] status codes.

## When not to use microservices

1. A modular monolith will be faster (no network serialization hops).
2. Too expensive to migrate (if migrating, use [[Strangler Fig]]).
3. Too difficult to control and observe without [[Distributed Tracing]] / [[OpenTelemetry]].
4. Not suitable for small teams/projects (Conway's Law overhead).
5. Strong cross-domain [[ACID]] [[Transaction|transactions]] matter (distributed transactions require [[2PC vs Saga|Saga]] / [[Outbox Pattern]]).
6. Hard to orchestrate containers without mature [[Containers orchestration]] ([[Kubernetes]]).

> [!warning] Failure modes & trade-offs
> - **The Distributed Monolith:** Splitting code into microservices that still share a database or require lockstep deployments gives you all the latency and operational cost of distributed systems with none of the independent deployability.
> - **Over-decomposition (nanoservices):** Too many tiny services turn a single user request into dozens of network hops, multiplying tail latency and failure modes (requiring [[Circuit Breaker]], [[Timeout Pattern]], and [[Retry Pattern]]).

## Flashcards

What is a "distributed monolith" anti-pattern in [[Microservices architecture]]?::Services deployed separately that remain tightly coupled (e.g. sharing a database or requiring coordinated lockstep releases), incurring distributed-system latency and complexity without independent deployability.

Decompose services along [[DDD]] bounded contexts with [[Database Per Service]]—not by single-table CRUD—to avoid chatty cross-service joins and distributed monoliths. #card-reminder

## Related
- [[Microservices architecture]]
- [[Microservices decomposition]]
- [[Database Per Service]]
- [[DDD]]
- [[2PC vs Saga]]
- [[Strangler Fig]]
- [[Service discovery]]
- [[Containers orchestration]]
