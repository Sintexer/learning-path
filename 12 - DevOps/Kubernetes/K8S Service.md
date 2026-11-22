---
flashcards:
  q-jipf: { nid: 1795356715268, hash: qa5fjduk, sync: tsc8zt7c }
  q-wkm2: { nid: 1795356715291, hash: gtr5ju9q, sync: m8djysta }
  q-e3vg: { nid: 1795356715325, hash: mg4i3egr, sync: nars9maj }
  q-zrgy: { nid: 1795356715347, hash: d97kehif, sync: rrni8pud }
  q-y4fw: { nid: 1795356715366, hash: jc3n4swu, sync: pxtbpvdx }
---

[[K8S Deployment]] runs [[Pod|Pods]] via [[K8S ReplicaSet|ReplicaSets]] to provide scaling and self-healing. However, clients cannot talk directly to Pod IPs—Pods are ephemeral and get new IPs whenever they restart or reschedule. That is where **Services** come in.

A **Kubernetes Service** is a stable network abstraction (virtual IP + [[K8S DNS|DNS name]] + port) that provides L4 ([[TCP]] and [[UDP]]) [[Load Balancing]] across a dynamic set of ready Pods matched by a [[K8S Label|label selector]]. As Pods scale up, die, or fail readiness probes, the **EndpointSlice** controller updates the Service's backing endpoints and `kube-proxy` (iptables/IPVS) or eBPF (Cilium) updates node routing rules automatically.

## Service Types

| Type | Exposure | Use Case |
| --- | --- | --- |
| **`ClusterIP`** (default) | Internal cluster virtual IP | Service-to-service communication inside the cluster |
| **`Headless` (`clusterIP: None`)** | No virtual IP; [[K8S DNS]] returns ready Pod IPs directly | StatefulSets ([[Kafka]], [[PostgreSQL]]), client-side [[gRPC]] LB |
| **`NodePort`** | Opens a static port (30000–32767) on every node | Dev/testing or underpinning cloud LoadBalancers |
| **`LoadBalancer`** | Provisions a cloud L4/L7 LB (AWS NLB/ALB) | Exposing services externally in cloud clusters |
| **`ExternalName`** | Returns a DNS `CNAME` to an external hostname | Aliasing external services into cluster DNS |

Because Services operate at L4 ([[OSI 4 layer]]), they do not inspect [[HTTP]] paths, hosts, or headers. For L7 HTTP/gRPC routing, [[TLS]] termination, and canary traffic splitting, use the **Kubernetes Gateway API** (or legacy Ingress) or a [[Service Mesh]].

> [!warning] Failure modes & trade-offs
> - **L4 load-balancing skew with [[gRPC]] / [[HTTP2]]**: `ClusterIP` distributes *connections*, not *requests*, so a long-lived HTTP/2/gRPC connection pins all RPCs to a single Pod. Fix with a [[Service Mesh]] (Envoy/Linkerd), L7 Gateway, or client-side load balancing over a Headless Service.
> - **Async `EndpointSlice` / `kube-proxy` propagation race**: when a Pod terminates, `kube-proxy` rules take seconds to remove the Pod IP across all nodes—combine readiness probes with a `preStop` sleep (see [[JVM App Deployment]]).
> - **`externalTrafficPolicy: Cluster` vs `Local`**: `Cluster` (default) SNATs external traffic across nodes (hiding the real client source IP and adding an extra hop); `Local` preserves client IP and avoids extra hops, but can cause uneven load if Pods aren't spread across all nodes.

## Flashcards

K8S Service:::A stable virtual IP, port, and [[K8S DNS]] name providing L4 ([[TCP]]/[[UDP]]) [[Load Balancing]] across a dynamic set of ready [[Pod|Pods]] selected by [[K8S Label|labels]].
^q-jipf

What are the four main Kubernetes Service types (plus Headless)?::==`ClusterIP`== (internal virtual IP), ==`NodePort`== (port on every node), ==`LoadBalancer`== (cloud LB), ==`ExternalName`== (DNS CNAME), and ==Headless (`clusterIP: None`)== (returns Pod IPs directly via DNS).
^q-e3vg
^q-wkm2

Why does a standard `ClusterIP` Service fail to balance [[gRPC]] traffic evenly across Pods?::Because `kube-proxy` balances at ==L4 (per TCP connection)==, while [[gRPC]] multiplexes all RPCs onto one persistent [[HTTP2]] connection; use a [[Service Mesh]] or client-side load balancing with a Headless Service.
^q-y4fw
^q-zrgy

## Related
- [[K8S DNS]]
- [[K8S Label]]
- [[Pod]]
- [[Load Balancing]]
- [[Service Mesh]]
- [[gRPC]]
- [[K8S Worker nodes]]
