---
flashcards:
  q-m9xv: { nid: 1795361678707, hash: fijenskt, sync: 43pq4w43 }
  q-6ddc: { nid: 1795361678778, hash: k7uwxzcs, sync: ikpgp63r }
  q-7kiy: { nid: 1795356715942, hash: pc648vsr, sync: ey9muc5i }
---

A [[Kubernetes]] *control plane node* is a server running the collection of system services that make up the brains of the cluster (sometimes called *Masters* or *Head nodes*).

Simple local setups run a single control plane node, while production environments run **3 or 5** control plane nodes for high [[Availability|availability]] (HA)—odd numbers for [[etcd]] [[Quorum Consensus|quorum]]. Best practice is to taint control plane nodes so user workloads do not compete for control plane CPU/RAM.

## API Server (`kube-apiserver`)

It is the Grand Central Station of Kubernetes. **All communication between all components goes through the API server**—nothing else talks directly to [[etcd]].

It exposes a [[REST|RESTful]] API over [[HTTPS]] where clients ([[kubectl]], controllers, CI/CD) submit YAML/JSON *manifests* describing the [[Pod Desired State|desired state]]. Every request passes through:
1. **[[Authentication]]** (X.509 certs, OIDC/[[JWT]], ServiceAccount tokens)
2. **[[Authorization]]** (RBAC, Node authorizer)
3. **Admission Controllers** (mutating webhooks -> schema validation -> validating webhooks / Pod Security Admission)
4. **Persistence to [[etcd]]** and fan-out via **watch streams** to controllers and `kubelet`.

Because `kube-apiserver` is stateless, it scales horizontally behind an L4/L7 [[Load Balancing|load balancer]].

## Cluster Store ([[etcd]])

The cluster store is the only *stateful* part of the control plane and persistently stores the entire configuration and state of the cluster—no store, no cluster.

It is backed by [[etcd]], a strongly consistent key-value store using [[Raft]]. Run **3 or 5** etcd members across separate Availability Zones for HA.

## Controllers and Controller Manager (`kube-controller-manager`)

The Controller Manager bundles and runs core control loops (Deployment, ReplicaSet, StatefulSet, Node, EndpointSlice, Job controllers). In HA clusters, replicas use **leader election** (`Lease` objects) so only one active controller manager reconciles at a time.

> The aim of a controller is to ensure the *observed state* of the cluster matches the *desired state*.

Every controller follows the same declarative [[Reconciliation|reconciliation loop]]:
1. Observe desired state (from API server watch/informer cache)
2. Observe current state
3. Diff desired vs current
4. Reconcile differences via API calls

Each controller is narrowly scoped (Unix philosophy), allowing complex cluster behavior to emerge from small, composable loops.

> Terms like *controller*, *control loop*, *watch loop*, and *reconciliation loop* refer to this same pattern.

## Scheduler (`kube-scheduler`)

Watches the [[#API Server]] for unscheduled [[Pod|Pods]] (`spec.nodeName == ""`) and binds each Pod to the best healthy [[K8S Worker nodes|worker node]] in two phases:
1. **Filtering (Predicates)**: eliminates nodes lacking CPU/RAM requests, matching taints/tolerations, node selectors, or volume topology.
2. **Scoring (Priorities)**: ranks remaining nodes (e.g., `topologySpreadConstraints`, bin-packing vs spreading, affinity) and binds the highest-scoring node.

If no node passes filtering, the Pod stays `Pending`.

## Cloud Controller Manager (`cloud-controller-manager`)

Decouples cloud-specific control loops (provisioning cloud [[Load Balancing|LoadBalancers]], attaching EBS/PD volumes, checking cloud VM node status) from core Kubernetes.

> [!warning] Failure modes & trade-offs
> - **Control plane outage ≠ instant data-plane outage**: if `kube-apiserver` or [[etcd]] goes down, existing healthy [[Pod|Pods]] keep serving traffic via existing `kube-proxy` rules—**but** no self-healing, autoscaling, rescheduling, or EndpointSlice updates can happen until the control plane recovers.
> - **[[etcd]] quorum loss**: losing 2 of 3 control plane nodes breaks [[Raft]] quorum; the cluster freezes writes until quorum is restored or rebuilt from snapshot.
> - **Broken validating/mutating webhooks**: a misconfigured admission webhook with `failurePolicy: Fail` whose backend pod is down can deadlock all Pod creations—even preventing recovery pods from starting!

## Flashcards

What are the four core components of the [[Kubernetes]] control plane?::==`kube-apiserver`== (gateway + auth/admission), ==`etcd`== (Raft state store), ==`kube-controller-manager`== (reconciliation loops), and ==`kube-scheduler`== (Pod-to-node binding).
^q-frc3
^q-m9xv

What happens to already-running user [[Pod|Pods]] if the Kubernetes control plane goes down temporarily?::Existing healthy Pods ==continue serving traffic== (data plane keeps running), but no new scheduling, self-healing, autoscaling, or EndpointSlice updates occur until the control plane recovers.
^q-hmrf
^q-6ddc

### What are the two phases `kube-scheduler` uses to place a Pending Pod onto a worker node? #card

1. **Filtering**: removes infeasible nodes (insufficient CPU/memory requests, unmatched taints, nodeSelectors, port/volume constraints).
2. **Scoring**: ranks feasible nodes by policies (affinity, `topologySpreadConstraints`, resource balance) and binds the Pod to the highest-scoring node.
^q-7kiy

## Related
- [[Kubernetes]]
- [[etcd]]
- [[K8S Worker nodes]]
- [[Pod Desired State]]
- [[K8S Version Skew]]
- [[Raft]]
- [[Reconciliation]]
