---
flashcards:
  q-9gfj: { nid: 1795361669007, hash: uarpbnsv, sync: qxwsyc9j }
  q-pmk7: { nid: 1795361669079, hash: pc2qp7tw, sync: iqfptkfq }

---

Worker nodes are the servers (physical hosts or cloud [[Virtual machine|VMs]]) that run user application [[Pod|Pods]] in a [[Kubernetes]] cluster.

Every worker node continuously:
1. Watches the [[K8S Control Plane#API Server|API server]] for Pods scheduled to it.
2. Runs containers, mounts volumes, and executes [[Health Checks|probes]].
3. Reports node conditions (`Ready`, `MemoryPressure`, `DiskPressure`, `PIDPressure`) and Pod status back to the [[K8S Control Plane]].

## Kubelet

The **kubelet** is the primary Kubernetes node agent running on every node (as a host systemd service, not a Pod).

When a node joins a cluster, `kubelet` registers its allocatable CPU, memory, ephemeral storage, and max Pods with the control plane and sends periodic heartbeats (`Lease` objects in `kube-node-lease`). If a node stops heartbeating, the Node controller taints it and evicts/reschedules its Pods onto healthy nodes.

## Container Runtime

`kubelet` delegates image pulling and container lifecycle to a CRI-compliant runtime—most commonly [[containerd]] or CRI-O—over the [[Container Runtime Interface|CRI]] [[gRPC]] API (which in turn invokes an [[Open Container Initiative|OCI]] runtime like `runc`).

## Kube-proxy & CNI

- **CNI plugin** (Calico, Cilium, AWS VPC CNI, Flannel): assigns each [[Pod]] a unique IP on the flat cluster Pod network and enforces `NetworkPolicy`.
- **`kube-proxy`**: watches [[K8S Service|Services]] and `EndpointSlices` from the API server and programs local **iptables**, **IPVS**, or **nftables** rules on every node to route and L4 load-balance `ClusterIP`/`NodePort` traffic to healthy Pod IPs. *(Note: eBPF-based CNIs like Cilium can replace `kube-proxy` entirely for better performance at scale.)*

> [!warning] Failure modes & trade-offs
> - **Node resource exhaustion & eviction**: always reserve `kube-reserved` and `system-reserved` CPU/memory on worker nodes so runaway user Pods cannot starve `kubelet` or [[containerd]] and flip the entire node to `NotReady`.
> - **`iptables` O(N) scaling limit**: in huge clusters (thousands of Services), legacy `kube-proxy` iptables mode evaluates rules linearly and slows down packet forwarding and rule sync—switch to **IPVS**, **nftables**, or **Cilium eBPF**.

## Flashcards

What are the three core agents/components running on a standard [[Kubernetes]] worker node?::==`kubelet`== (node agent managing Pods/probes), a ==[[Container Runtime Interface|CRI]] container runtime== (e.g., [[containerd]]), and ==`kube-proxy`== (plus a CNI plugin for Pod networking).
^q-vjtg
^q-9gfj

What is the difference between the CNI plugin and `kube-proxy` on a worker node?::The **CNI plugin** assigns ==Pod IPs== and enforces `NetworkPolicy`; **`kube-proxy`** programs node-level L4 routing rules (iptables/IPVS/nftables) for ==[[K8S Service|Services]]==.
^q-upau
^q-pmk7

## Related
- [[K8S Control Plane]]
- [[Container Runtime Interface]]
- [[containerd]]
- [[K8S Service]]
- [[Pod]]
- [[K8S Version Skew]]
