---
flashcards:
  q-eq74: { nid: 1795356714395, hash: i4wayn9m, sync: vxw267mk }
  q-jmb5: { nid: 1795356714416, hash: j5u9dfkg, sync: nqt8572m }
---

> Pronounced *container dee*

**containerd** is an industry-standard, CNCF-graduated, high-level [[Container]] runtime used by [[Kubernetes]] (via [[Container Runtime Interface|CRI]]) and also used under the hood by Docker Engine itself.

The distinction between Docker and `containerd` is the difference between a full developer platform and a focused execution engine:
- **Docker** bundles image building (`buildx`), developer CLI, Compose, and networking into a large user-facing suite (which itself delegates execution down to `containerd` -> `runc`).
- **`containerd`** strips away the build engine and developer UX to focus strictly on the machine-centric container lifecycle: pulling/pushing [[Open Container Initiative|OCI]] images, managing image snapshots, and spawning/supervising containers via low-level OCI runtimes (`runc`, `crun`, Kata, gVisor).

```text
kubelet --(CRI gRPC)--> containerd --(OCI runtime-spec)--> runc --> container process
```

Kubernetes standardized on `containerd` (and CRI-O) because removing the extra Docker daemon + `dockershim` layers lowers resource overhead, shrinks the security attack surface, and simplifies operational debugging (`crictl` / `ctr`).

> [!tip] Production reality
> On a [[K8S Worker nodes|Kubernetes worker node]] running `containerd`, `docker ps` does not exist—use **`crictl ps`** / **`crictl logs`** (CRI-level) or **`ctr -n k8s.io containers list`** when debugging a broken node where `kube-apiserver` is unreachable.

## Flashcards

containerd:::CNCF-graduated, high-level [[Container]] runtime that implements [[Kubernetes]] [[Container Runtime Interface|CRI]] for image management and container lifecycle, delegating low-level execution to [[Open Container Initiative|OCI]] runtimes like `runc`.
^q-eq74

What is the execution chain from `kubelet` down to a running Linux container process on a modern Kubernetes node?::`kubelet` --([[Container Runtime Interface|CRI]])--> `containerd` --([[Open Container Initiative|OCI]])--> `runc` --> Linux container process.
^q-jmb5

## Related
- [[Container Runtime Interface]]
- [[Open Container Initiative]]
- [[Container]]
- [[K8S Worker nodes]]
- [[Kubernetes]]
