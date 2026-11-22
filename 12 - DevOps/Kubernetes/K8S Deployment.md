---
flashcards:
  q-mpfh: { nid: 1795356715617, hash: 4sh2avmj, sync: 9tp3ycfu }
  q-5vgw: { nid: 1795356715650, hash: eg8ax9aq, sync: 45prj44z }
  q-6qdt: { nid: 1795356715669, hash: zrerfi34, sync: 9un62d7a }
  q-w4a3: { nid: 1795356715692, hash: c8bbmzms, sync: x4pyukz8 }
  q-p64n: { nid: 1795356715725, hash: 63cn3tsz, sync: 8ie7gkn8 }
---

A **Deployment** is a higher-level [[Kubernetes]] object that wraps around [[K8S ReplicaSet|ReplicaSets]] and [[Pod|Pods]] to provide declarative self-healing, [[Horizontal scaling|scaling]], [[Zero-downtime deployment|zero-downtime rollouts]], and versioned rollbacks for **stateless** applications.

Most of the time you deploy Pods indirectly via higher-level controllers:
- **Deployment**: stateless apps (web APIs, workers).
- **StatefulSet**: stateful apps needing stable network identity and persistent storage per replica (e.g., [[Kafka]], [[PostgreSQL]]).
- **DaemonSet**: one Pod per node (log shippers, [[OpenTelemetry]] collectors, CNI agents).
- **Job / CronJob**: finite batch tasks.

Behind the scenes, each is implemented as a [[K8S Control Plane#Controllers and Controllers Manager|Controller]] running a reconciliation loop to keep observed state aligned with [[Pod Desired State|desired state]].

There are two major pieces to a Deployment:
1. **The spec**: declarative YAML describing desired replicas, selector ([[K8S Label]]), Pod template, and rollout strategy.
2. **The controller**: reconciles ReplicaSets to match the spec.

A Deployment manages a single Pod template. For example, an app with a front-end web service and a back-end catalog needs two separate Deployments, each managing its own set of identical [[K8S ReplicaSet|replicas]].

## Rolling Update

A **Rolling Update** is the default Deployment strategy in Kubernetes, incrementally scaling up a new [[K8S ReplicaSet]] while scaling down the old one so traffic never drops.

Key rollout parameters:
- `maxUnavailable`: maximum Pods allowed offline during the update (default `25%`).
- `maxSurge`: maximum extra Pods allowed above desired replicas during the update (default `25%`).
- `minReadySeconds`: minimum seconds a newly Ready Pod must stay healthy before the rollout proceeds.
- `progressDeadlineSeconds`: marks a stuck rollout as failed if no progress occurs within the deadline (default `600s`).
- `revisionHistoryLimit`: how many old ReplicaSets to retain for `kubectl rollout undo` (default `10`).

During the rollout, Kubernetes checks each new Pod's **readiness probe** ([[Health Checks]]). A new Pod receives [[K8S Service]] traffic only after passing readiness, and only then does Kubernetes terminate an old Pod. If the new version crashes (`CrashLoopBackOff`) or fails readiness, the rollout pauses automatically while old healthy Pods keep serving traffic, allowing an instant `kubectl rollout undo`.

> [!warning] Failure modes & trade-offs
> - **Zero-downtime still requires app & Pod hygiene**: a Rolling Update alone will drop requests unless you configure readiness probes, graceful shutdown + `preStop` sleep (see [[JVM App Deployment]]), and a **`PodDisruptionBudget` (PDB)** to protect against voluntary node drains during upgrades.
> - **Mixed-version overlap**: `v1` and `v2` run concurrently during the rollout—APIs, messages, and DB schemas must be [[Backward Compatibility|backward compatible]] (see [[Zero-downtime deployment]]).
> - **Missing `resources.requests/limits`**: without CPU/memory requests, the scheduler overpacks nodes and HPA cannot compute utilization percentages.

## Flashcards

What is the hierarchy of objects managed by a [[Kubernetes]] Deployment?::`Deployment` manages ==`ReplicaSets`== (one per revision), which manage ==`Pods`== (which run containers).
^q-5vgw
^q-mpfh

In a Kubernetes Rolling Update, what do `maxSurge` and `maxUnavailable` control?::`maxSurge` limits how many ==extra Pods above desired replicas== can be created during rollout; `maxUnavailable` limits how many ==Pods can be unavailable== simultaneously.
^q-w4a3
^q-6qdt

### Why is a Kubernetes `Deployment` with Rolling Update not sufficient by itself for zero-downtime upgrades? #card

You also need accurate **readiness probes** (so booting Pods don't get traffic), a **`preStop` sleep + graceful `SIGTERM` handler** (to beat the async `EndpointSlice`/`kube-proxy` update race), a **`PodDisruptionBudget`** (for node drains), and [[Backward Compatibility|backward-compatible]] APIs/schemas during `v1`/`v2` overlap.
^q-p64n

## Related
- [[K8S ReplicaSet]]
- [[Pod]]
- [[Zero-downtime deployment]]
- [[JVM App Deployment]]
- [[K8S Service]]
- [[K8S Label]]
- [[Health Checks]]
