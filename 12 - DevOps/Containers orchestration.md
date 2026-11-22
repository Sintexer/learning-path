---
flashcards:
  q-nmkm: { nid: 1795356716667, hash: jy3mvmzg, sync: ve49nh7h }
  q-pgrb: { nid: 1795356716691, hash: z52aj349, sync: 6bfqumq5 }
---

Although running one [[Container]] instance is uncomplicated to manage, it lacks high [[Availability|availability]] and [[Scaling|scalability]]. Most organizations run hundreds or thousands of containers across fleets of hosts.

When managing compute at scale, you must solve:
- **Scheduling & bin-packing**: how to place containers onto nodes based on CPU/memory requests, affinity, and topology spread across Availability Zones.
- **Self-healing**: what happens when a container crashes or an entire host dies (automated restarts via [[Health Checks]] and rescheduling).
- **[[Zero-downtime deployment|Rollouts & rollbacks]]**: rolling updates, canary shifts, and instant rollback on regression.
- **[[Service discovery]] & [[Load Balancing]]**: routing traffic dynamically as ephemeral container IPs churn.
- **Configuration, secrets, and autoscaling**: injecting config from an [[External Configuration Store]] and scaling replicas horizontally ([[Horizontal scaling]]).

This coordination is handled by a **container orchestration platform**. The industry standard is [[Kubernetes|K8S]] (often managed via [[AWS Elastic Kubernetes Service|Amazon EKS]], GKE, or AKS). Cloud providers also offer simpler native orchestrators like [[AWS Elastic Container Service|Amazon ECS]] (which pairs cleanly with [[AWS Fargate]]).

> [!tip] Production reality
> Orchestration is not free: [[Kubernetes]] gives portability, a rich ecosystem ([[Service Mesh]], operators, GitOps), and multi-team standardization, but carries significant control-plane, networking, and upgrade overhead ([[K8S Version Skew]]). Small teams on a single cloud often ship faster on managed ECS/Fargate or Cloud Run until they genuinely need Kubernetes primitives.

## Flashcards

Container orchestration::Automated management of container scheduling, self-healing, [[Scaling|scaling]], [[Service discovery]], networking, and rolling deployments across a cluster of hosts.
^q-nmkm

### What core operational problems does a container orchestrator solve? #card

- Bin-packing and placement across nodes/AZs.
- Self-healing (restarting failed containers and rescheduling off dead nodes via [[Health Checks]]).
- [[Zero-downtime deployment|Rolling updates and rollbacks]].
- Dynamic [[Service discovery]], [[Load Balancing]], secret injection, and autoscaling.
^q-pgrb

## Related
- [[Container]]
- [[Kubernetes]]
- [[K8S Control Plane]]
- [[Cloud-Native App]]
- [[Zero-downtime deployment]]
- [[Service discovery]]
