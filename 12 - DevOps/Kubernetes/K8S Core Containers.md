These four system images are the bare minimum typically seen in a lightweight [[Kubernetes]] (K3s) cluster:

- **`coredns` (The Phonebook — [[K8S DNS]]):** Because [[Pod]] IP addresses churn constantly as Pods die and restart, CoreDNS resolves stable [[K8S Service]] names (e.g., `orders.prod.svc.cluster.local`) to ClusterIPs or Pod IPs.
- **`local-path-provisioner` (Local Storage Provisioner):** When a stateful workload (like [[PostgreSQL]]) requests a PersistentVolumeClaim (PVC) in a local cluster, this provisioner dynamically binds a directory on the node's disk (in cloud clusters, a CSI driver provisions EBS/PD volumes instead).
- **`metrics-server` (Resource Telemetry):** Scrapes `kubelet` summary APIs for live CPU and memory usage, powering `kubectl top nodes/pods` and the **Horizontal Pod Autoscaler (HPA)**. (It is *not* a long-term metrics DB like Prometheus / [[OpenTelemetry]].)
- **`pause` (The Pod Sandbox Container):** Starts first in every [[Pod]]. Its sole job is to hold open the shared Linux namespaces (network IP/ports, IPC, optionally PID) so all application containers in the Pod can share `localhost` and survive individual container restarts without losing the Pod IP.

> [!tip] Production reality
> If `metrics-server` is missing or broken, **HPA stops scaling** even if your app is pegged at 100% CPU. And if `CoreDNS` crashes or gets overloaded, almost every internal service call fails with DNS resolution timeouts—always run at least 2 CoreDNS replicas spread across nodes.

## Flashcards

What is the purpose of the `pause` container inside every [[Kubernetes]] [[Pod]]?::It starts first and holds open the Pod's shared ==Linux network and IPC namespaces== (keeping the Pod IP alive even if an application container crashes and restarts).

Which core Kubernetes component is required for both `kubectl top` and Horizontal Pod Autoscaler (HPA) CPU/memory scaling?::==`metrics-server`==.

## Related
- [[Kubernetes]]
- [[K8S DNS]]
- [[Pod]]
- [[Running Kubernetes Cluster Locally]]
- [[K8S Worker nodes]]
