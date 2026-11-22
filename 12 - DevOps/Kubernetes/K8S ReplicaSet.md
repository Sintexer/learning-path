Behind the scenes, [[K8S Deployment|Deployments]] rely on another object called a **ReplicaSet**. While you should almost never create or manage ReplicaSets directly (let the Deployment controller manage them), understanding their role explains how rollouts and rollbacks actually work.

At a high level:
- **[[Container|Containers]]** package applications and dependencies.
- **[[Pod|Pods]]** provide the shared execution/network sandbox on a node.
- **ReplicaSets** maintain a stable number of identical Pod replicas (self-healing and [[Horizontal scaling|scaling]]), matching Pods via [[K8S Label|label selectors]].
- **[[K8S Deployment|Deployments]]** manage ReplicaSets over time—creating a new ReplicaSet for each Pod template change to orchestrate rolling updates and retaining old ReplicaSets (scaled to 0) for instant `kubectl rollout undo`.

> [!tip] Production reality
> Each ReplicaSet gets a unique `pod-template-hash` label injected by the Deployment controller so Pods from old and new revisions never collide in the same ReplicaSet. Note also that ReplicaSets track Pod *ownership* via `metadata.ownerReferences`—deleting a Deployment cascades deletion to its ReplicaSets and Pods.

## Flashcards

What is the difference between a [[K8S ReplicaSet|ReplicaSet]] and a [[K8S Deployment|Deployment]]?::A **ReplicaSet** ensures N identical [[Pod|Pods]] are running (scaling + self-healing); a **Deployment** manages multiple ReplicaSets to add ==rolling updates, revision history, and rollbacks==.

How does `kubectl rollout undo` roll back a Deployment so quickly?::The Deployment retains previous ==ReplicaSets== (scaled to 0 replicas, up to `revisionHistoryLimit`) and simply scales the old ReplicaSet back up while scaling the broken one down.

## Related
- [[K8S Deployment]]
- [[Pod]]
- [[K8S Label]]
- [[Kubernetes]]
- [[Horizontal scaling]]
