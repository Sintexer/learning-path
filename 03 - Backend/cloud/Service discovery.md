---
flashcards:
  q-d86a: { nid: 1795356733691, hash: qtkpuiu3, sync: g8vtvgjy }
  q-p57g: { nid: 1795356733716, hash: 6bdnc2ie, sync: ebvd2bzu }
---

# Service Discovery

**Service discovery** is the mechanism by which services in a dynamic cloud or [[Microservices architecture|microservices]] environment automatically locate the network addresses (`IP:port`) of other services without hardcoding hostnames or IPs.

In containerized and autoscaled platforms ([[Kubernetes]], AWS ECS), service instances are ephemeral—IPs change on every deploy, crash, or scale event.

## Core Components

1. **Service Registry:** A highly available directory mapping logical service names to healthy instance endpoints (e.g. Consul, Netflix Eureka, ZooKeeper, or [[etcd]] inside [[Kubernetes]]).
2. **Registration:**
   - **Self-registration:** Each instance registers itself on startup and sends periodic heartbeats.
   - **Third-party registration:** The orchestrator (e.g. [[K8S Control Plane]]) watches container lifecycle and updates the registry automatically.
3. **[[Health Checks]]:** Active probes (HTTP/TCP/gRPC readiness probes) or TTL heartbeats remove unhealthy instances from routing.

## Discovery Patterns

### 1. Client-Side Discovery
- The calling service queries the service registry (and caches the instance list locally), then picks an instance using a client-side [[Load Balancing|load balancer]] (e.g. Spring Cloud LoadBalancer + Eureka/Consul).
- **Pros:** Fewer network hops; client can use custom load-balancing strategies (zone-aware, [[Consistent Hashing]]).
- **Cons:** Couples every language/service to a discovery SDK.

### 2. Server-Side Discovery (Platform / Proxy)
- The client sends requests to a well-known virtual IP / DNS name (e.g. [[K8S Service]] via [[K8S DNS]], AWS ALB, or a [[Service Mesh]] sidecar like Envoy). The router queries the registry and forwards the request.
- **Pros:** Language-agnostic; clients stay simple.
- **Cons:** Extra network hop / proxy infrastructure to operate.

> [!warning] Failure modes & trade-offs
> - **Stale endpoints during rolling deploys:** Graceful shutdown races (pod stops accepting traffic before kube-proxy/DNS/registry propagates removal) cause transient `502`/`Connection Refused` errors. Combine `preStop` sleep hooks, readiness probes, and idempotent [[Retry Pattern|retries]].
> - **DNS TTL & JVM caching:** [[DNS]]-based discovery can pin traffic to dead IPs if clients cache DNS lookups too long (or over persistent [[HTTP2]] / [[gRPC]] connections, which bypass L4 load balancing unless you use L7 / client-side / [[Service Mesh]] load balancing).
> - **Registry CAP trade-off:** Strongly consistent (CP) registries like [[etcd]] or ZooKeeper refuse writes during a [[Split-Brain]] / quorum loss (preferring accuracy), while AP registries like Eureka continue serving cached/possibly stale instances (preferring [[Availability]]).

## Flashcards

## What is the difference between Client-Side and Server-Side Service Discovery? #card
In **Client-Side Discovery**, the caller queries the service registry directly and runs load-balancing logic in-process (fewer hops, but SDK coupling per language). In **Server-Side Discovery**, the caller talks to a router/VIP (e.g. [[K8S Service]] or load balancer) that queries the registry and forwards traffic (language-agnostic, extra hop).

Why does L4 [[Load Balancing]] / [[DNS]] service discovery unevenly distribute [[gRPC]] or [[HTTP2]] traffic?::[[HTTP2]]/[[gRPC]] multiplexes many requests over a single long-lived [[TCP]] connection, so L4 balancing pins all requests to the single backend chosen at connection setup; use L7 load balancing, a [[Service Mesh]], or client-side discovery.
^q-d86a

## How do CP registries (etcd/ZooKeeper) differ from AP registries (Eureka) during a network partition? #card
CP registries prioritize consistency—they reject registrations/updates without quorum to prevent [[Split-Brain]]. AP registries (Eureka self-preservation mode) prioritize [[Availability]]—they keep serving cached instance lists even if heartbeats stop, tolerating brief staleness rather than dropping all routes.
^q-p57g

## Related
- [[K8S Service]]
- [[K8S DNS]]
- [[Load Balancing]]
- [[Health Checks]]
- [[Service Mesh]]
- [[Microservices Sidecar pattern]]
- [[CAP Theorem]]
- [[etcd]]
- [[Zero-downtime deployment]]
