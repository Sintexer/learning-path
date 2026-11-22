A microservices app is built from lots of independent small specialized parts that work together to form a meaningful application. For example, you might have an e-commerce app that comprises all of the following small specialized components:
- Web front-end
- Catalog service
- Shopping cart
- Authentication service
- Logging service
- Persistent store (ideally [[Database Per Service]])

Each of these individual services is called a microservice. Typically, each is coded and owned by a different team. Each can have its own release cycle and can be scaled independently ([[Horizontal scaling]]). For example, you can patch and scale the logging microservice without affecting any of the others.

Building applications this way is vital for [[Cloud-Native App]] features, and is explored in depth in [[Microservices architecture]] and [[Microservices decomposition]].

> [!warning] Failure modes & trade-offs
> - **Distributed monolith**: if services share a database or require synchronized lockstep deployments, you pay all the network/operational tax of microservices with none of the independent deployability.
> - **Latency & partial failure**: in-process function calls become network RPCs ([[HTTP]], [[gRPC]]) that can time out, fail partially, or cascade unless protected by [[Timeout Pattern|timeouts]], [[Circuit Breaker|circuit breakers]], and [[Bulkhead Pattern|bulkheads]].
> - **Data consistency**: cross-service transactions lose single-DB [[ACID]] guarantees and require [[Saga|Sagas]], [[Outbox Pattern]], and [[Eventual Consistency]].

## Flashcards

What is the defining operational benefit of a microservices application over a monolith?::Independent deployment, scaling, and team ownership per bounded service without coordinating releases across the whole application.

A microservices system that shares one database or requires lockstep deployments across services is called a ==distributed monolith==.

## What new failure modes appear when splitting a monolith into microservices? #card

Network latency and partial failures, cascading outages across synchronous call chains, loss of single-database [[ACID]] transactions (requiring [[Saga|Sagas]] / [[Eventual Consistency]]), and harder debugging (requiring [[Distributed Tracing]] / [[OpenTelemetry]]).

## Related
- [[Microservices architecture]]
- [[Microservices decomposition]]
- [[Cloud-Native App]]
- [[Database Per Service]]
- [[Microservices Resilience and Fault tolerance]]
- [[Distributed Tracing]]
