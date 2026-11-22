---
flashcards:
  q-6dns: { nid: 1795356714591, hash: un4ur5ed, sync: q2ke7x4d }
  q-42d8: { nid: 1795356714617, hash: u86iyy98, sync: v6nxwzky }
  q-j3d8: { nid: 1795356714642, hash: wxpp5yan, sync: xxipxu7z }
  q-k4h2: { nid: 1795356714667, hash: h2r3g3id, sync: zk9a28r8 }
---

Multi-container [[Pod|Pods]] are a powerful pattern when two or more containers must be tightly coupled, co-scheduled on the same node, and share `localhost` networking, volumes, or lifecycle. Each container still keeps a single clear responsibility.

## Core Multi-Container Pod Patterns

1. **Sidecar pattern** (see [[Microservices Sidecar pattern]]): extends or enhances the main container without changing app code—e.g., an Envoy [[Service Mesh]] proxy handling mTLS/retries/telemetry, a log forwarder tailing a shared volume, or a config/secret syncer.
2. **Adapter pattern**: transforms or standardizes the main container's output for an external consumer—e.g., translating legacy app metrics into Prometheus / [[OpenTelemetry]] format.
3. **Ambassador pattern**: proxies outbound connections from the main container to external systems—e.g., a local Cloud SQL Auth Proxy or [[Redis]]/sharding proxy listening on `localhost` so the app stays agnostic to connection topology.
4. **Init container pattern**: runs one or more *init containers* sequentially to completion **before** any main application container starts (e.g., waiting for a dependency, fetching secrets, or seeding a shared volume). If an init container fails, Kubernetes restarts the Pod until it succeeds.

## Native Sidecar Containers (K8s 1.28+)

Historically, regular sidecars caused two painful production bugs:
- **Startup race**: the main app started *before* the sidecar proxy (e.g., Envoy / Vault Agent) was ready, failing early requests.
- **Job hang**: in `Job` / `CronJob` Pods, when the main container exited `0`, the sidecar kept running forever so the Job never completed.

Kubernetes **native sidecars** solve both: define the sidecar under `initContainers` with `restartPolicy: Always`. Kubernetes starts it before the main app, waits for its readiness probe, keeps it running alongside the app, and **automatically terminates it after the main container exits**.

> [!warning] Failure modes & trade-offs
> - **Don't bundle independent services in one Pod**: containers in a Pod always scale together and share fate—only co-locate helper containers that genuinely share lifecycle and `localhost`/volume with the main container; otherwise use separate [[K8S Deployment|Deployments]].

## Flashcards

What are the four classic [[Kubernetes]] multi-container [[Pod]] patterns?::==Sidecar== (extends functionality, e.g., proxy/log shipper), ==Adapter== (normalizes output), ==Ambassador== (proxies outbound connectivity), and ==Init container== (runs to completion before main containers start).
^q-42d8
^q-6dns

How do Kubernetes native sidecar containers (K8s 1.28+) fix startup races and hanging `Job` pods?::They are declared in `initContainers` with ==`restartPolicy: Always`==, so they start before the main container and automatically terminate once the main container finishes.
^q-k4h2
^q-j3d8

## Related
- [[Pod]]
- [[Microservices Sidecar pattern]]
- [[Microservices Service mesh pattern]]
- [[Service Mesh]]
- [[OpenTelemetry]]
