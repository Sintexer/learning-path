---
aliases:
  - K8S
flashcards:
  q-fs4t: { nid: 1795356714992, hash: tr9vrtiq, sync: 37c9hwbt }
  q-7z2i: { nid: 1795361666833, hash: tz4nqcnd, sync: n925dg59 }

---
> v1.0 since 2015, donated to CNCF in 2014; pronounced *koo-ber-net-eez* (or K8s as *kates*).

**Kubernetes** is the industry-standard open-source [[Containers orchestration|container orchestration]] platform. It deploys, scales, self-heals, and updates [[Container|containerized]] [[Cloud-Native App|cloud-native applications]] declaratively across a fleet of machines.

Kubernetes and container runtimes are complementary technologies: Kubernetes handles high-level orchestration (scheduling, self-healing, [[Horizontal scaling|scaling]], [[Zero-downtime deployment|rolling updates]], [[Service discovery]]), while delegating low-level container execution to a [[Container Runtime Interface|CRI]] runtime such as [[containerd]] or CRI-O (and [[Open Container Initiative|OCI]] runtimes like `runc` or Kata).

> Docker (`dockershim`) was deprecated in Kubernetes 1.20 and removed in 1.24 in favor of [[containerd]] / CRI-O, though standard Docker-built OCI images continue to run identically.

Kubernetes abstracts infrastructure the way an OS abstracts hardware—developers declare *what* desired state they want rather than *which* machine should run each process.

## Kubernetes as a Cluster

A Kubernetes cluster consists of two planes:
- **[[K8S Control Plane]]**: the cluster brains (`kube-apiserver`, [[etcd]], `kube-controller-manager`, `kube-scheduler`).
- **[[K8S Worker nodes]]**: the machines (`kubelet`, [[containerd]], `kube-proxy`, CNI) where user application [[Pod|Pods]] run.

## Packaging Apps for Kubernetes

To run an application on Kubernetes:
1. Package it as an [[Open Container Initiative|OCI]] [[Container]] image.
2. Wrap it in a [[Pod]] specification with [[Health Checks|probes]], resource requests/limits, and graceful shutdown (see [[JVM App Deployment]]).
3. Declare its [[Pod Desired State|desired state]] in a manifest (YAML / Helm / Kustomize) via a higher-level controller such as a [[K8S Deployment]] (stateless) or StatefulSet (stateful), exposed via a [[K8S Service]].

> [!tip] Production reality
> Kubernetes is a **reconciliation engine**, not an imperative deployment script: every controller continuously converges observed state toward desired state. That design gives strong self-healing, but also means misconfigurations (bad liveness probes, missing CPU/memory limits, missing `PodDisruptionBudget`) can trigger automated restart or eviction loops at cluster speed.

## Flashcards

Kubernetes (K8s):::Declarative [[Containers orchestration|container orchestration]] platform composed of a [[K8S Control Plane|control plane]] and [[K8S Worker nodes|worker nodes]] that continuously reconciles observed cluster state toward desired state.
^q-fs4t

What architectural pattern sits at the core of Kubernetes?::A declarative ==reconciliation (control) loop== where specialized controllers watch the API server and continuously converge observed state to match [[Pod Desired State|desired state]] stored in [[etcd]].
^q-mnds
^q-7z2i

## Related
- [[K8S Control Plane]]
- [[K8S Worker nodes]]
- [[Pod]]
- [[K8S Deployment]]
- [[K8S Service]]
- [[etcd]]
- [[Containers orchestration]]
- [[Cloud-Native App]]
