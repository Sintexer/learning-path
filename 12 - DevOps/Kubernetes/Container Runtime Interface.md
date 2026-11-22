---
aliases:
  - CRI
flashcards:
  q-isgf: { nid: 1795356715967, hash: zw9p3bdj, sync: mh2rfqt2 }
  q-27g5: { nid: 1795356715991, hash: tvcbgs72, sync: qnjvj7nw }
  q-ht7d: { nid: 1795356716025, hash: bb2ttm55, sync: grctwq3j }
  q-726b: { nid: 1795356716047, hash: t4gvb86a, sync: uw5ek2qx }
  q-2wsy: { nid: 1795356716066, hash: 8p6g6ubv, sync: w5cmz7vm }
---

**Container Runtime Interface (CRI)** is a [[gRPC]] plugin API that decouples the [[K8S Worker nodes#Kubelet|kubelet]] from any specific [[Container]] runtime implementation in [[Kubernetes]].

## How CRI Works

Instead of hardcoding Docker internals inside `kubelet`, `kubelet` acts as a [[gRPC]] client speaking two CRI services over a local Unix domain socket to any CRI-compliant runtime:
- **`RuntimeService`**: manages [[Pod]] sandboxes (starting the `pause` container, setting up namespaces/CNI networking) and container lifecycle (`Create`, `Start`, `Stop`, `Remove`, `Exec`, stats).
- **`ImageService`**: pulls, lists, and removes [[Open Container Initiative|OCI]] images from container registries.

```text
kubelet --(CRI over gRPC)--> containerd / CRI-O --(OCI runtime-spec)--> runc / Kata / gVisor
```

## Common CRI Implementations

- **[[containerd]]**: CNCF graduated runtime, default in most managed Kubernetes clusters (EKS, GKE, AKS, K3s).
- **CRI-O**: lightweight runtime built specifically for Kubernetes (default in OpenShift).
- **Sandboxed runtimes** (via `RuntimeClass`): **Kata Containers** (microVMs) and **gVisor** (user-space kernel) for stronger multi-tenant isolation.

> [!tip] Production reality
> Kubernetes removed `dockershim` in **v1.24**—not because Docker images stopped working (they follow the [[Open Container Initiative|OCI]] image spec), but because Docker Engine did not natively implement CRI, forcing Kubernetes to maintain a brittle translation shim (`kubelet -> dockershim -> dockerd -> containerd -> runc`). Dropping `dockershim` removed two unnecessary hops.

## Flashcards

CRI (Container Runtime Interface):::The [[gRPC]] API between `kubelet` and a high-level container runtime ([[containerd]], CRI-O) that abstracts Pod sandbox, container, and image lifecycle in [[Kubernetes]].
^q-isgf

What are the two [[gRPC]] services exposed by the Container Runtime Interface (CRI)?::==`RuntimeService`== (Pod sandbox and container lifecycle) and ==`ImageService`== (pulling, listing, and removing OCI images).
^q-ht7d
^q-27g5

Why did Kubernetes remove `dockershim` in v1.24 without breaking existing Docker images?::Docker Engine did not implement CRI natively (requiring an extra shim in front of [[containerd]]), while Docker images are standard ==[[Open Container Initiative|OCI]] images== that [[containerd]] and CRI-O run directly.
^q-2wsy
^q-726b

## Related
- [[containerd]]
- [[Open Container Initiative]]
- [[K8S Worker nodes]]
- [[Kubernetes]]
- [[Container]]
- [[gRPC]]
