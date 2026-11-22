---
flashcards:
  q-j3d2: { nid: 1795361673233, hash: jpu94im6, sync: ihjgjsh9 }
  q-825y: { nid: 1795361673304, hash: vdi8z2bu, sync: vii2ab9w }
  q-shhs: { nid: 1795356715592, hash: cw9b8nq2, sync: jyjeve66 }
---

**Namespaces** are a native way to partition a single [[Kubernetes]] cluster into multiple *virtual clusters* for organizing teams, environments, RBAC permissions ([[Authorization]]), `ResourceQuota`, and `LimitRange` policies.

Most Kubernetes objects are **namespaced** (e.g., [[Pod|Pods]], [[K8S Service|Services]], [[K8S Deployment|Deployments]], ConfigMaps, Secrets). Cluster-scoped objects exist outside namespaces (e.g., Nodes, PersistentVolumes, Namespaces, ClusterRoles, StorageClasses). If no namespace is specified, resources land in the `default` namespace.

System namespaces include `kube-system` (control plane & [[K8S Core Containers|CoreDNS/metrics-server]]), `kube-public`, and `kube-node-lease` (node heartbeats).

> [!warning] Failure modes & trade-offs
> - **Namespaces are NOT strong security or network isolation by default**: by default, a Pod in namespace `dev` can freely connect to any Pod in namespace `prod` over the flat Pod network! Enforce isolation explicitly with **`NetworkPolicy`** (supported by Cilium/Calico), **RBAC**, and **Pod Security Admission (PSA)** (which replaced the removed `PodSecurityPolicy` in K8s 1.25).
> - **Soft vs hard multi-tenancy**: namespaces share the same control plane, `kube-apiserver`, and worker node kernels—for untrusted or regulated hard multi-tenancy, use separate clusters (or virtual clusters + sandboxed runtimes).
> - **Stuck `Terminating` namespace**: usually caused by a leftover `finalizer` on a custom resource whose controller was deleted first.

## Flashcards

Are [[Kubernetes]] Namespaces isolated from each other at the network layer by default?::==No==—the Pod network is flat across namespaces by default; you must apply **`NetworkPolicy`** rules (with a supporting CNI like Calico/Cilium) to restrict cross-namespace traffic.
^q-2yn5
^q-j3d2

What replaced the deprecated/removed `PodSecurityPolicy` in Kubernetes 1.25+ for enforcing Pod security standards per namespace?::==Pod Security Admission (PSA)== (enforcing `privileged`, `baseline`, or `restricted` profiles via namespace labels).
^q-sn5p
^q-825y

Namespaces provide logical grouping, RBAC boundaries, and `ResourceQuota` limits—not hard kernel/control-plane isolation for untrusted tenants. #card-reminder
^q-shhs

## Related
- [[Kubernetes]]
- [[K8S DNS]]
- [[Pod]]
- [[Authorization]]
- [[Least privilege principle]]
- [[Zero-Trust Security]]
