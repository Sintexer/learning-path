
> **Central question:** How should clients access a system composed of many services without becoming tightly coupled to its internal structure?

An **API Gateway** provides a controlled entry point.


Suppose a browser displays an order details page. It needs data from 4 services, so it must know how to call each of them. That introduces problems, such as:
- The browser must know internal service addresses and API contracts.
- Every service needs appropriate external-access protection.
- The client must combine responses and handle partial failures.
- Mobile clients may make several requests over a high-latency network.
- Internal service changes can force client changes.

There are two distinct needs here:
1. **Control access to services.**
2. **Shape data for a client.**

Those lead to Gateway and [[BFF]] respectively.

An API Gateway receives external requests and routes them to the appropriate backend.

Common responsibilities include:
1. Routing
2. Authentication
3. Rate limiting
4. Requests limits (oversized requests rejects)
5. Protocol adaptation
6. Observability
7. API versioning support

##  Gateway Retries Can Duplicate Business Operations

Gateway request failure - retry might produce duplicated operation (instead of trying to place the same order, gateway might submit two distinct orders).A safe design uses a stable request [[Idempotancy Key]]:

```http
POST /orders
Idempotency-Key: checkout-operation-123
```

The backend associates that key with the logical operation and its result. Any Gateway retry must preserve the same key.

# 8.10. When These Patterns Become a Problem

## A. The Gateway becomes a business monolith

Warning signs:

- Complex order workflows inside routing filters.
- Shared business rules for every domain.
- Every feature requires a Gateway deployment.
- Gateway changes require coordination across all teams.

Keep edge policy separate from domain behavior.

## B. Too many BFFs

A BFF per screen or minor frontend variation creates duplicated code and excessive deployments.

Create separate BFFs when clients have meaningfully different needs or ownership—not merely because there are two user interfaces.

## C. Shared libraries recreate tight coupling

A shared authentication utility can be useful.

A giant shared BFF library containing all customer, order, and payment rules can force coordinated releases across otherwise independent applications.

## D. The Gateway is treated as automatically highly available

A logical central entry point need not be a single physical instance.

Run redundant instances, load balance them, and account for:

- Capacity.
- Configuration failures.
- Certificate failures.
- Authentication-provider outages.
- Downstream connection exhaustion.

Centralizing traffic also concentrates operational responsibility.

---

# 8.11. Gateway vs. Reverse Proxy vs. Service Mesh

These terms overlap, but their emphasis differs:

| Term              | Primary emphasis                                               |
| ----------------- | -------------------------------------------------------------- |
| [[Reverse proxy]] | Forwarding requests on behalf of servers                       |
| API Gateway       | API-facing routing, security, quotas, and policy               |
| [[BFF]]           | Client-specific API adaptation and aggregation                 |
| [[Service mesh]]  | Service-to-service traffic security, policy, and observability |

A Gateway is commonly implemented using reverse-proxy capabilities. A service mesh generally addresses **[[IT Common Glossary#East-West Traffic|east-west traffic]]** between services. A Gateway commonly handles **north-south traffic** entering or leaving the application boundary.

Neither a mesh nor a Gateway provides Saga correctness, business authorization, or idempotency automatically.