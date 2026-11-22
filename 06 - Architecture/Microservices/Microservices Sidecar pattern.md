**What is the Sidecar Pattern?** The sidecar pattern involves deploying a helper [[Container|container]] (sidecar) alongside the main application container within the same [[Pod]] (in [[Kubernetes]], see [[Multi-container Pods]]) or deployment unit. They share the same network namespace (`localhost`) and lifecycle, letting the sidecar extend the main application without modifying its code.

## Use Cases

1. **[[Microservices Service mesh pattern|Service Mesh]]**: Sidecars (e.g., Envoy) act as proxies to handle east-west communication, mTLS ([[TLS]]), and observability ([[Service Mesh]]).
2. **Logging and Telemetry**: Sidecars (e.g., Fluent Bit, [[OpenTelemetry]] Collector) ship logs, metrics, and [[Distributed Tracing|traces]] off the host.
3. **[[External Configuration Store|Configuration & Secrets]]**: Sidecars (e.g., Vault Agent) fetch and rotate secrets/config files on a shared in-memory volume.
4. **Security & Auth**: Sidecars handle [[Authentication]], [[Authorization]], or mTLS termination transparently.

## Benefits

- **Polyglot Modularity**: Enhances the main application across any language (Java, Go, Python, Node) without duplicating a fat [[Shared Library]].
- **Independent Patching**: Security/proxy CVEs can be patched by updating the sidecar image without rebuilding application code.
- **Isolation**: Keeps the main application focused on business logic while offloading infrastructure concerns.

## Challenges

- **Resource Overhead**: Every replica runs an extra container, multiplying memory and CPU overhead across large clusters.
- **Lifecycle & Startup Races**: If the main app starts making network calls before the sidecar proxy is ready (or if a Batch Job finishes and the sidecar keeps the [[Pod]] running forever), pods fail or hang.
- **Debugging Complexity**: Network hops through `iptables` / local proxy add latency and complicate troubleshooting.

> [!tip] Production reality
> - **Kubernetes Native Sidecars (K8s 1.28+)**: Define sidecars as `initContainers` with `restartPolicy: Always` so Kubernetes guarantees the sidecar starts **before** the main container and terminates cleanly **after** Jobs finish.
> - **Sidecar vs. DaemonSet vs. eBPF**: For high-density clusters, teams increasingly move heavy proxies or log shippers out of per-pod sidecars into node-level DaemonSets or sidecarless eBPF data planes (e.g., Istio Ambient, Cilium) to cut memory overhead.

Sidecar Pattern:::Deploying an auxiliary container alongside the main application container in the same Pod (sharing network and storage) to offload cross-cutting concerns like mTLS, logging, or config rotation without modifying app code.

## Why is the Sidecar pattern often preferred over a Shared Library in polyglot microservices, and what is its main cost? #card
A sidecar provides uniform mTLS, telemetry, or secret rotation across **any programming language** and can be patched without recompiling apps. Its main cost is **per-pod resource overhead** (CPU/memory multiplied by every replica) and extra container lifecycle/networking complexity.

## What classic Kubernetes lifecycle bugs do Native Sidecars (`initContainers` + `restartPolicy: Always`) solve? #card
1. **Startup race**: App starts sending requests before the proxy/secret sidecar is ready.
2. **Job hang**: A one-off Kubernetes `Job` container finishes, but the long-running sidecar prevents the Pod from ever terminating.

## Related
- [[Multi-container Pods]]
- [[Pod]]
- [[Kubernetes]]
- [[Microservices Service mesh pattern]]
- [[Service Mesh]]
- [[Shared Library]]
- [[External Configuration Store]]
- [[OpenTelemetry]]
