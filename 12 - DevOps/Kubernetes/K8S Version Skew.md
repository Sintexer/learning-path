---
flashcards:
  q-enn6: { nid: 1795361658582, hash: zydu8fvr, sync: bwp4gfaa }
  q-9gu4: { nid: 1795361658653, hash: e3mizvbk, sync: 9iz5wn8r }
  q-ga9r: { nid: 1795361658729, hash: 7vienq52, sync: zpgkjh3v }

---

[[Kubernetes]] enforces a strict **Version Skew Policy** defining the maximum minor-version difference allowed between control plane components, worker nodes, and CLI clients during rolling cluster upgrades.

## Supported Skew Matrix (relative to `kube-apiserver`)

| Component | Allowed Version Skew vs `kube-apiserver` | Notes |
| --- | --- | --- |
| **`kube-apiserver` (HA peers)** | At most **1 minor** version apart | During HA control plane rollout |
| **`kube-controller-manager` / `kube-scheduler` / `cloud-controller-manager`** | Same minor or **up to 1 minor older** | Must never be newer than `kube-apiserver` |
| **`kubelet`** | Same minor or **up to 3 minor older** (K8s 1.28+) | Must never be newer than `kube-apiserver` |
| **`kube-proxy`** | Up to **3 minor older or newer** than `kubelet` (and <= `kube-apiserver`) | Must not be newer than `kube-apiserver` |
| **`[[kubectl]]`** | **±1 minor version** (1 newer or 1 older) | e.g., a 1.36 client safely talks to 1.35, 1.36, or 1.37 |

## Mandatory Upgrade Order

1. Upgrade **control plane** first (`kube-apiserver` -> `kube-controller-manager` -> `kube-scheduler`), **one minor version at a time** (never skip minor versions, e.g., 1.33 -> 1.35 directly is unsupported).
2. Upgrade **worker nodes** (`kubelet` / `kube-proxy`) via rolling node replacement (cordon -> drain respecting `PodDisruptionBudget` -> replace).

> [!warning] Failure modes & trade-offs
> - **Removed API versions**: Kubernetes regularly deprecates and removes beta/older API versions (e.g., `Ingress`, `CronJob`, `PodSecurityPolicy`). Before upgrading a minor version, audit live manifests and Helm charts (`pluto` / `kubent`) so deployments don't silently fail post-upgrade.
> - **Skipping minor versions**: jumping two minor versions at once can corrupt stored [[etcd]] schemas or break control-plane version skew guarantees.

## Flashcards

What is the supported [[Kubernetes]] version skew for `[[kubectl]]` vs `kube-apiserver`?::±==1 minor version== (one minor version newer or older than the API server).
^q-idwt
^q-enn6

How many minor versions older than `kube-apiserver` can `kubelet` be (since K8s 1.28)?::Up to ==3 minor versions older== (and `kubelet` must never be newer than `kube-apiserver`).
^q-8n28
^q-9gu4

In what order must you upgrade a Kubernetes cluster?::Upgrade the ==control plane== first (one minor version at a time), then upgrade ==worker nodes== (`kubelet` / `kube-proxy`).
^q-f7v6
^q-ga9r

## Related
- [[Kubernetes]]
- [[K8S Control Plane]]
- [[K8S Worker nodes]]
- [[kubectl]]
- [[etcd]]
