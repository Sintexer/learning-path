> **Central question:** How should clients access a system composed of many services without becoming tightly coupled to its internal structure?

An **API Gateway** provides a controlled entry point.

Suppose a browser displays an order details page. It needs data from 4 services, so it must know how to call each of them. That introduces problems, such as:
- The browser must know internal service addresses and [[API]] contracts.
- Every service needs appropriate external-access protection ([[Authentication]] and [[Authorization]]).
- The client must combine responses and handle partial failures.
- Mobile clients may make several requests over a high-latency network.
- Internal service changes can force client changes.

There are two distinct needs here:
1. **Control access to services.**
2. **Shape data for a client.**

Those lead to API Gateway and [[BFF]] respectively.

An API Gateway receives external requests and routes them to the appropriate backend (often integrating with [[Service discovery]] and [[Load Balancing]]).

Common responsibilities include:
1. Routing
2. [[Authentication]] / [[JWT]] validation
3. [[Rate Limiting]] and quotas
4. Request limits (rejecting oversized payloads)
5. Protocol adaptation (e.g., [[REST]] / [[HTTP]] to [[gRPC]])
6. Observability ([[Distributed Tracing]], access logs, metrics)
7. [[API versioning]] support

## Gateway Retries Can Duplicate Business Operations

If a Gateway request fails or times out, an automatic [[Retry Pattern|retry]] might produce a duplicated operation (instead of trying to place the same order, the gateway might submit two distinct orders). A safe design uses a stable request [[Idempotency Key]]:

```http
POST /orders
Idempotency-Key: checkout-operation-123
```

The backend associates that key with the logical operation and its result (see [[API Idempotency]]). Any Gateway retry must preserve the same key.

## When These Patterns Become a Problem

### A. The Gateway becomes a business monolith

Warning signs:
- Complex order workflows inside routing filters.
- Shared business rules for every domain.
- Every feature requires a Gateway deployment.
- Gateway changes require coordination across all teams.

Keep edge policy separate from domain behavior.

### B. Too many BFFs

A [[BFF]] per screen or minor frontend variation creates duplicated code and excessive deployments.

Create separate BFFs when clients have meaningfully different needs or ownership — not merely because there are two user interfaces.

### C. Shared libraries recreate tight coupling

A shared authentication utility can be useful.

A giant [[Shared Library|shared BFF library]] containing all customer, order, and payment rules can force coordinated releases across otherwise independent applications.

### D. The Gateway is treated as automatically highly available

A logical central entry point need not be a single physical instance.

Run redundant instances, load balance them, and account for:
- Capacity.
- Configuration failures.
- [[TLS]] certificate failures.
- [[Identity provider|Authentication-provider]] outages.
- Downstream connection exhaustion.

Centralizing traffic also concentrates operational responsibility.

> [!warning] Failure modes & trade-offs
> Because **every** north-south request passes through the Gateway, a bad regex route, misconfigured [[Rate Limiting]] rule, or slow auth lookup causes a **total platform outage**. Keep Gateway logic thin, stateless, and fast; push domain orchestration and client-specific shaping down into [[BFF|BFFs]] or domain services.

---

## Gateway vs. Reverse Proxy vs. Service Mesh

These terms overlap, but their emphasis differs:

| Term | Primary emphasis |
| --- | --- |
| [[Reverse proxy]] | Forwarding requests on behalf of servers |
| API Gateway | API-facing routing, security, quotas, and policy |
| [[BFF]] | Client-specific API adaptation and aggregation |
| [[Service Mesh]] ([[Microservices Service mesh pattern]]) | Service-to-service traffic security, policy, and observability |

A Gateway is commonly implemented using reverse-proxy capabilities. A [[Service Mesh]] generally addresses **[[IT Common Glossary#East-West Traffic|east-west traffic]]** between services. A Gateway commonly handles **north-south traffic** entering or leaving the application boundary.

Neither a mesh nor a Gateway provides [[Saga]] correctness, business authorization, or [[Idempotency]] automatically.

## How do an API Gateway, a BFF, and a Service Mesh differ in responsibility? #card
- **API Gateway**: Edge control for **north-south traffic** (routing, auth, [[Rate Limiting]], TLS, quotas).
- **[[BFF]]**: Shapes and aggregates backend APIs for a specific client type (web vs mobile).
- **[[Service Mesh]]**: Handles **east-west** service-to-service networking (mTLS, retries, observability, traffic splitting) without domain logic.

Why must an API Gateway avoid embedding business workflows?::It turns the edge into a deployment bottleneck and single point of coordination/failure across all teams; keep edge policy separate from domain logic.

An API Gateway retry on a non-idempotent POST can duplicate business actions unless the client and backend enforce an ==Idempotency Key==. #card-reminder

## Related
- [[BFF]]
- [[Aggregator]]
- [[Reverse proxy]]
- [[Microservices Service mesh pattern]]
- [[Service Mesh]]
- [[Idempotency Key]]
- [[API Idempotency]]
- [[Rate Limiting]]
- [[API versioning]]
