**Labels** are key-value metadata pairs (`app: payments`, `tier: backend`, `env: prod`) used to **loosely couple** [[Kubernetes]] resources.

Each time a [[Pod]] is created from a [[K8S Deployment]], it receives labels from `spec.template.metadata.labels`. Controllers ([[K8S ReplicaSet|ReplicaSets]], Deployments) and [[K8S Service|Services]] discover their target Pods dynamically via **label selectors** (`spec.selector.matchLabels` or `matchExpressions`). Kubernetes rejects a Deployment manifest at validation time if `spec.selector` does not match `spec.template.metadata.labels`.

## Labels vs Annotations

- **Labels**: **indexed, identifying metadata** used for querying, selecting, grouping, and routing (`kubectl get pods -l app=payments`, Service routing, NetworkPolicy targeting, pod affinity).
- **Annotations**: **non-identifying metadata** not used for selection—used for tooling config, Prometheus/[[OpenTelemetry]] scrape settings, Ingress controller directives, git commit SHAs, or change causes.

> [!warning] Failure modes & trade-offs
> - **Immutable Deployment selectors**: `spec.selector.matchLabels` on a [[K8S Deployment]] is immutable after creation; changing it requires recreating the Deployment.
> - **Overly broad Service selectors**: if a [[K8S Service]] selects only `app: api` and you spin up a canary or debug Pod with `app: api`, production traffic immediately routes to that Pod! Always include distinct version/track or release labels when isolating traffic.
> - **Removing a label from a live Pod** detaches it from its ReplicaSet (which immediately spins up a replacement Pod) while leaving the unlabeled Pod running for live debugging/forensics—a super handy production trick!

## Flashcards

What is the difference between [[Kubernetes]] **Labels** and **Annotations**?::**Labels** are indexed key-value pairs used to ==select and loosely couple resources== (Services, ReplicaSets, NetworkPolicies); **Annotations** store ==non-identifying metadata== for tooling, config, or build info.

How can you isolate a misbehaving [[Pod]] from production traffic for live debugging without killing it?::Change or remove the label matched by its [[K8S Service]] and [[K8S ReplicaSet]]—the Service stops routing traffic to it and the ReplicaSet spawns a healthy replacement, while the broken Pod stays alive for inspection.

## Related
- [[K8S Service]]
- [[K8S Deployment]]
- [[K8S ReplicaSet]]
- [[Pod]]
- [[Kubernetes]]
