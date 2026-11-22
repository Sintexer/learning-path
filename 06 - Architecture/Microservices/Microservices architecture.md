---
flashcards:
  q-k5gi: { nid: 1795356717567, hash: 8978ikcy, sync: yvhpe7d8 }
  q-5jgc: { nid: 1795356717592, hash: aig7br6j, sync: hq6vn9pc }
---

The microservices approach is commonly used in modern application development. This includes [[Serverless|serverless]] applications, where your application's software is constructed of multiple loosely coupled services. Microservices architectures reduce the complexity of the individual component, but introduce a different type of complexity at scale related to the number of independent components and network boundaries.

Microservices architecture is a design approach where a single application is composed of multiple loosely coupled and independently deployable services (see [[Microservices App]]). Each service is responsible for a specific business capability ([[DDD|bounded context]]) and can be developed, deployed, and scaled independently. This approach contrasts with monolithic architecture, where all functionalities are tightly integrated into a single deployment unit.

## Key Characteristics

- **Decentralization**: Each service can use different technologies and databases ([[Database Per Service]]).
- **Autonomy**: Teams can develop, deploy, and scale services independently.
- **Resilience**: Failure in one service does not necessarily affect others (when paired with [[Microservices Resilience and Fault tolerance]]).
- **[[Scaling|Scalability]]**: Services can be scaled independently based on demand ([[Horizontal scaling]]).

## Pros and Cons of Microservices Architecture

**Pros:**

1. **Scalability**: Individual services can be scaled independently, allowing more efficient use of resources.
2. **Flexibility in Technology**: Different services can use different technologies best suited for their needs.
3. **Resilience**: Failures in one service are isolated and do not necessarily bring down the entire system.
4. **Faster Development and Deployment**: Smaller, independent teams can develop, test, and deploy services more quickly.
5. **Improved Maintainability**: Smaller codebases are easier to understand, maintain, and refactor.

**Cons:**

1. **Complexity**: Managing multiple services introduces complexity in deployment, monitoring, and communication.
2. **Data Management**: Ensuring data consistency across services is much harder without local [[ACID]] transactions (requires [[Saga]], [[Outbox Pattern]], [[CQRS]]).
3. **Inter-Service Communication**: Network calls add latency, serialization overhead, and partial failure modes.
4. **Operational Overhead**: Requires mature DevOps, CI/CD, observability, and [[Containers orchestration|orchestration]] practices.
5. **Testing**: End-to-end testing is harder due to distributed state and version skew.

## Comparison with Monolithic Architecture

**Monolithic Architecture:**
- **Single Codebase & Deployment**: All functionalities ship together as one unit.
- **Tight Coupling (if unstructured)**: Harder to scale individual modules independently, though a **modular monolith** can keep clean internal boundaries without network overhead.
- **Simpler Development**: Easier to develop, refactor, and test in early stages; supports simple local [[ACID]] transactions.

**Microservices Architecture:**
- **Multiple Codebases & Independent Deployment**: Services ship on independent cadences.
- **Loose Coupling**: Services communicate over explicit [[API]] or event contracts.
- **Higher Operational Floor**: Pays off when organizational scale (multiple teams) or independent scaling/isolation needs justify the distributed-systems tax.

> [!warning] Failure modes & trade-offs
> **The Distributed Monolith**: Splitting a codebase into microservices while keeping shared databases, synchronous chatty call chains, or giant [[Shared Library|shared domain libraries]] gives you the **worst of both worlds** — the tight deployment coupling of a monolith *plus* the network latency, partial failures, and operational tax of distributed systems. Start with a **modular monolith** unless team autonomy, independent scaling, or fault-isolation boundaries clearly demand separate services (see [[Microservices decomposition]]).

## Key Areas of Complexity & Core Patterns

See the full index in [[Microservices patterns]].

1. **[[Service discovery|Service Discovery]]**
   - **Client-Side Discovery**: Clients query a service registry to find network locations.
   - **Server-Side Discovery**: A load balancer / [[K8S Service]] routes requests to healthy instances.
   - **Tools**: Kubernetes DNS ([[K8S DNS]]), Consul, Eureka.

2. **[[API Gateway]] & [[BFF]]**
   - **Responsibilities**: Routing, [[Authentication]], [[Rate Limiting]], protocol adaptation, client-specific aggregation ([[Aggregator]]).
   - **Tools**: Spring Cloud Gateway, Kong, Envoy Gateway.

3. **Inter-Service Communication**
   - **Synchronous**: [[HTTP]]/[[REST]], [[gRPC]].
   - **Asynchronous**: Message queues (RabbitMQ, SQS), event streams ([[Kafka]]).

4. **Data Management**
   - **[[Database Per Service]]**: Exclusive schema ownership; polyglot persistence.
   - **[[Eventual Consistency]]**: [[Outbox Pattern]], [[Change Data Capture]], [[CQRS]], [[Event Sourcing]], and [[Saga]] ([[2PC vs Saga]]).

5. **[[Microservices Resilience and Fault tolerance|Resilience and Fault Tolerance]]**
   - [[Timeout Pattern]], [[Circuit Breaker]], [[Retry Pattern]], [[Bulkhead Pattern]], [[Fallback Pattern]], [[Health Checks]], [[Backpressure]].
   - **Tools**: Resilience4j (modern Java/Spring standard; Hystrix is deprecated).

6. **[[Event-driven architecture|Event-Driven Architecture]]**
   - Producers, consumers, brokers ([[Kafka]]), [[Delivery Semantics]], [[Idempotent Consumer]], [[DLQ]].

7. **[[Microservices Orchestration and Choreography|Service Orchestration vs. Choreography]]**
   - Centralized workflow control ([[Temporal]], [[Saga State Table]]) vs. decentralized event reactions.

8. **Security in Microservices**
   - [[Authentication and Authorization]] ([[OIDC]], [[JWT]]), [[HTTPS]]/[[TLS]], east-west mTLS via [[Service Mesh]], [[Zero-Trust Security]].

9. **Observability (Logging, Metrics, Tracing)**
   - Centralized structured logs, golden-signal metrics (Prometheus/Grafana), and [[Distributed Tracing]] / [[Microservices Tracing]] ([[OpenTelemetry]], Jaeger, Tempo).

10. **Containerization and Orchestration**
   - [[Container|Containers]] (Docker/[[containerd]]), [[Containers orchestration|orchestration]] ([[Kubernetes]]), [[Microservices Sidecar pattern|sidecars]], and [[Microservices Service mesh pattern|Service Mesh]] (Istio, Linkerd).

Distributed Monolith:::An anti-pattern where a system is split into separately deployed microservices that remain tightly coupled (via shared DBs, synchronous call chains, or shared domain libraries), paying distributed-systems overhead without gaining independent deployability.
^q-k5gi

## When should you prefer a Modular Monolith over Microservices? #card
When a small/medium team does not yet have independent scaling, fault-isolation, or multi-team organizational boundaries to justify the distributed-systems tax (network latency, [[Saga|Sagas]], [[Eventual Consistency]], observability, and [[Kubernetes]] operational overhead). You can always extract services later via [[Strangler Fig]].

Microservices reduce ==single-component code complexity== while increasing ==distributed operational and data-consistency complexity==.
^q-5jgc

## Related
- [[Microservices patterns]]
- [[Microservices decomposition]]
- [[Microservices Resilience and Fault tolerance]]
- [[Microservices Orchestration and Choreography]]
- [[Database Per Service]]
- [[DDD]]
- [[Strangler Fig]]
- [[Kubernetes]]
