---
flashcards:
  q-56p2: { nid: 1795356729843, hash: ah8u2kua, sync: mk94tfj4 }
  q-xjg3: { nid: 1795356729866, hash: xmu9jdje, sync: pe5fznk4 }
  q-knzk: { nid: 1795356729892, hash: m75s7z9m, sync: eetbu8bq }
  q-7vzg: { nid: 1795356729918, hash: gz9chvf7, sync: a3zs2ex7 }
---

A **load balancer** distributes incoming network traffic across multiple backend instances to improve throughput, [[Availability|availability]], and fault isolation.

## L4 vs L7 Load Balancing

- **L4 (Transport layer — [[OSI 4 layer]])**: routes based on IP and [[TCP]]/[[UDP]] port without inspecting payloads. Very fast and low latency, but cannot route by URL path/headers and struggles to rebalance long-lived [[HTTP2]]/[[gRPC]] connections.
- **L7 (Application layer — [[OSI 7 layer]])**: acts as a [[Reverse proxy]], terminates [[TLS]], inspects [[HTTP]] path/host/headers/cookies, and routes per-request. Required for content-based routing, [[HTTP2]]/[[gRPC]] request multiplexing, and canary splits.

## What are the common load-balancing algorithms?

### Static Algorithms

1. **Round robin** — client requests are sent to different service instances in sequential order. Services are usually required to be stateless.
2. **Sticky round-robin** — an improvement of round-robin where a client's subsequent requests are pinned to the same backend instance (via cookie or IP affinity).
3. **Weighted round-robin** — the admin assigns weights to each backend; higher-weight instances handle proportionally more requests (useful for heterogeneous hardware or canary rollouts).
4. **Hash** — applies a hash function on incoming IP, URL, or key so identical keys hit the same backend (maximizes local cache hit rate). Use [[Consistent Hashing]] so adding/removing nodes only remaps a small fraction of keys.

### Dynamic Algorithms

1. **Least connections** — sends each new request to the instance with the fewest active connections (better when request durations vary widely).
2. **Least response time** — routes to the instance with the lowest latency (often combined with active connections, e.g., EWMA).
3. **Power of Two Choices (P2C)** — picks two random backends and routes to the less loaded one; avoids herd effects across distributed load balancers without global coordination.

> [!warning] Failure modes & trade-offs
> - **L4 + [[HTTP2]] / [[gRPC]] imbalance**: because HTTP/2 multiplexes many RPCs over a single persistent [[TCP]] connection, an L4 load balancer pins all requests from a client to one backend. Use L7 load balancing (or client-side load balancing / [[Service Mesh]]).
> - **Sticky sessions break failover & autoscaling**: pinning clients to one node causes uneven load ("hot spots") and drops sessions when that node restarts.
> - **Bad [[Health Checks]]**: if a health check calls a slow downstream DB, a DB hiccup marks *all* healthy app instances unhealthy simultaneously, causing a total outage.

## Flashcards

What is the key difference between L4 and L7 load balancing?::L4 routes at the [[TCP]]/[[UDP]] connection level (IP + port, fast, no payload inspection); L7 terminates [[TLS]]/[[HTTP]] and routes per-request using URL, headers, or host.
^q-56p2

Why does L4 load balancing fail to evenly distribute [[gRPC]] or [[HTTP2]] traffic?::Because HTTP/2 multiplexes many requests over a single long-lived [[TCP]] connection, so an L4 balancer pins all requests from that client to one backend; L7 or client-side load balancing is required.
^q-xjg3

Using ==[[Consistent Hashing]]== instead of modulo hashing ensures that adding or removing a backend node only remaps a minimal fraction of keys.
^q-knzk

## Why is Power of Two Choices (P2C) preferred over global least-connections in distributed load balancers? #card

With multiple independent load balancers, global least-connections suffers from stale metrics and "thundering herd" (all balancers pile onto the same newly-idle node). Sampling two random nodes and picking the less loaded one achieves near-optimal balance with O(1) work and no global state.
^q-7vzg

## Related
- [[Reverse proxy]]
- [[Horizontal scaling]]
- [[Active-active systems]]
- [[Health Checks]]
- [[Consistent Hashing]]
- [[Service Mesh]]
- [[HTTP2]]
- [[gRPC]]
