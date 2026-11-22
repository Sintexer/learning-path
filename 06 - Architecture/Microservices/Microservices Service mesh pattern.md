**What is a Service Mesh?** A [[Service Mesh]] is an infrastructure layer that manages **east-west** (service-to-service) communication within a [[Microservices architecture]]. It handles traffic management, security, resilience, and observability, abstracting these networking concerns out of application code (compare with [[API Gateway]], which handles north-south edge traffic).

## Key Components

- **Data Plane**: Lightweight network proxies (traditionally [[Microservices Sidecar pattern|sidecars]] like Envoy deployed alongside each service [[Pod]], or newer node-level/ambient proxies) that intercept all inbound and outbound service traffic.
- **Control Plane**: Manages and configures the data plane proxies — distributing routing rules, [[TLS]] certificates, authorization policies, and collecting telemetry.

## Features

1. **Traffic Management**: Intelligent routing, L7 [[Load Balancing]], canary splits, and [[Shadow Testing|traffic mirroring]].
2. **Security**: Automatic mutual [[TLS]] (**mTLS**) with short-lived certificate rotation, service identity (SPIFFE), and fine-grained authorization policies (enabling [[Zero-Trust Security]]).
3. **Observability**: Uniform [[Distributed Tracing]] ([[OpenTelemetry]]), golden-signal metrics (latency, error rate, throughput), and access logs across polyglot services.
4. **Resilience**: Infrastructure-level [[Circuit Breaker|outlier detection]], [[Retry Pattern|retries]], and [[Timeout Pattern|timeouts]].

## Popular Service Mesh Solutions

- **Istio**: Feature-rich open-source mesh (Envoy sidecars + newer sidecarless **Ambient Mode**).
- **Linkerd**: Lightweight, Rust-proxy mesh focused on simplicity, low overhead, and operational ease.
- **Consul**: Combines [[Service discovery]], [[External Configuration Store|configuration]], and service mesh capabilities.

## Benefits

- **Decoupling**: Separates operational networking concerns from application logic across polyglot stacks (avoiding fat [[Shared Library|shared libraries]]).
- **Consistency**: Enforces uniform mTLS, telemetry, and routing policies across every service.
- **Security**: Encrypts and authenticates all east-west traffic by default.

## Challenges

- **Operational Complexity**: Adds control-plane upgrades, certificate-authority management, and harder network debugging.
- **Resource & Latency Overhead**: Extra proxy hops add tail latency and memory/CPU overhead per pod (driving interest in eBPF/sidecarless designs like Cilium and Istio Ambient).

> [!warning] Failure modes & trade-offs
> - **Blind retries on non-idempotent APIs**: A mesh proxy only sees HTTP status codes, not business semantics — enabling automatic mesh retries on `POST` endpoints without [[Idempotency Key|idempotency keys]] can duplicate payments/orders.
> - **Control-plane / mTLS outage**: An expired root CA or broken control-plane config push can cut off *all* east-west traffic simultaneously.
> - **App vs Mesh double-resilience**: If both the app (Resilience4j) and mesh retry/timeout independently, budgets clash and amplify load. And note: a mesh does **not** solve [[Saga]] correctness, [[Idempotency]], or [[Distributed Tracing]] header propagation *inside* your app (your app must still forward `traceparent` from incoming to outgoing calls!).

Service Mesh (Data Plane vs. Control Plane):::Infrastructure layer for east-west service networking where the **Data Plane** (proxies) intercepts traffic for mTLS, routing, and telemetry, and the **Control Plane** distributes configs, policies, and certificates.

## What can a Service Mesh NOT do automatically for Distributed Tracing and Resilience? #card
1. **Tracing**: The proxy cannot correlate an inbound request to an outbound call inside your process — your application code must still propagate trace headers (`traceparent`).
2. **Resilience / Semantics**: The mesh does not know which endpoints are idempotent or handle business-level [[Fallback Pattern|fallbacks]]/[[Saga]] compensations; blind mesh retries on mutating calls can duplicate operations.

## Related
- [[Service Mesh]]
- [[Microservices Sidecar pattern]]
- [[API Gateway]]
- [[Zero-Trust Security]]
- [[TLS]]
- [[Distributed Tracing]]
- [[OpenTelemetry]]
- [[Shadow Testing]]
- [[Microservices Resilience and Fault tolerance]]
- [[Kubernetes]]
