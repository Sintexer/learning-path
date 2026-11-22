---
flashcards:
  q-rnev: { nid: 1795356714692, hash: rmivmpe8, sync: 6t2imff8 }
  q-u7d2: { nid: 1795356714717, hash: mqkq3u8p, sync: qfemtyi7 }
  q-eg4h: { nid: 1795356714741, hash: 2d67whmm, sync: hxwjva8m }
  q-vze2: { nid: 1795356714768, hash: ytsq33rk, sync: swxvs2bd }
---

The **desired state** is declared in YAML/JSON manifest files (or Helm/Kustomize templates) that tell [[Kubernetes]] *what* an application should look like—which [[Container]] image, how many replicas, resource requests/limits, ports, [[Health Checks|probes]], and rollout strategy.

The simplest way to submit a manifest to the API server is `[[kubectl]] apply -f` (or declaratively via GitOps tools like Argo CD / Flux). Once persisted in [[etcd]], unscheduled Pods are bound by the [[K8S Control Plane#Scheduler|scheduler]] to [[K8S Worker nodes|worker nodes]], where [[K8S Worker nodes#Kubelet|kubelet]] pulls images via [[Container Runtime Interface|CRI]], attaches networking/volumes, and starts the containers.

## The Declarative [[Reconciliation]] Loop

In [[Kubernetes]], the declarative model works like this:

1. Declare the desired state (`spec`) of an application in a manifest file.
2. Submit it to the [[K8S Control Plane#API Server|API Server]] (validated by auth/RBAC/admission controllers).
3. Kubernetes persists it to the [[K8S Control Plane#Cluster store|cluster store (etcd)]] as the single source of truth.
4. Specialized controllers (e.g., [[K8S Deployment]], [[K8S ReplicaSet]]) continuously watch observed state (`status`) vs desired state (`spec`).
5. Whenever drift occurs (a Pod crashes, a node dies, or `spec` changes), controllers take idempotent corrective actions until `status` converges to `spec`.

> [!tip] Production reality
> **Level-triggered vs edge-triggered**: Kubernetes controllers are **level-triggered** (they inspect current state vs desired state on every tick/watch), not edge-triggered (reacting only to one-off events). That makes the system resilient to missed events, controller restarts, and network partitions—the core reason Kubernetes self-heals reliably.

## Flashcards

What is the difference between `spec` and `status` on a [[Kubernetes]] resource?::`spec` is the user-declared ==desired state==; `status` is the ==observed current state== continuously updated by controllers and `kubelet`.
^q-u7d2
^q-rnev

Why are Kubernetes controllers designed to be **level-triggered** rather than edge-triggered?::Level-triggered loops compare current observed state against desired state directly, so ==missed events or controller restarts== self-correct on the next reconciliation cycle.
^q-vze2
^q-eg4h

## Related
- [[Kubernetes]]
- [[K8S Control Plane]]
- [[K8S Deployment]]
- [[Pod]]
- [[etcd]]
- [[Reconciliation]]
- [[kubectl]]
