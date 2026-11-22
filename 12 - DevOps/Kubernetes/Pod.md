---
flashcards:
  q-k34y: { nid: 1795356714792, hash: ninpe4kt, sync: qbz4rx6j }
  q-twvd: { nid: 1795356714817, hash: c2fzw4n5, sync: rzt86gmq }
  q-qtgj: { nid: 1795356714841, hash: kqfyutet, sync: x49tcajf }
  q-hxta: { nid: 1795356714868, hash: wiue6pum, sync: p569sy42 }
  q-84yu: { nid: 1795356714893, hash: r97bm5ni, sync: m84fvqaw }
---

A **Pod** is the smallest deployable and schedulable unit in [[Kubernetes]]—a sandbox that runs one or more tightly coupled [[Container|containers]]. Pods themselves don't execute application code; applications run inside containers hosted in the Pod's shared environment.

Pods ring-fence an area of the host OS via the `pause` container ([[K8S Core Containers]]), creating shared Linux namespaces so all containers in the Pod share:
- **Network namespace**: single Pod IP, port space, routing table—containers in the same Pod talk over `localhost`.
- **IPC namespace**: System V IPC and POSIX shared memory.
- **Volumes**: shared storage mounts (e.g., `emptyDir`, PVCs, ConfigMaps/Secrets).
- *(Optional)* **Shared PID namespace** (`shareProcessNamespace: true`).

> [!tip] Scaling
> If you need to scale an app, add or remove Pods ([[Horizontal scaling]]). You **never** scale by stuffing more app replicas into the same Pod.

Usually a Pod runs a single container. Advanced [[Multi-container Pods]] include [[Service Mesh]] sidecars (Envoy), log forwarders, and init containers.

## Key Pod Properties

- **Atomic scheduling**: a Pod is scheduled to a **single** [[K8S Worker nodes|worker node]]—you cannot split one Pod's containers across multiple nodes.
- **Ephemeral & immutable**: Pods live, die, and are replaced by new Pods with new IPs. You do not patch a running Pod's core spec in place; controllers delete and replace Pods instead.
- **Pod-level controls**: [[K8S Label|labels]] & annotations, restart policies (`Always`, `OnFailure`, `Never`), [[Health Checks|startup/readiness/liveness probes]], affinity/anti-affinity & `topologySpreadConstraints`, `terminationGracePeriodSeconds` + `preStop` hooks, `securityContext`, and resource `requests`/`limits`.

## Flat Pod Networking

Each Pod gets a unique IP in the flat cluster Pod network (assigned by the CNI plugin). Every Pod can reach every other Pod directly without NAT or port mapping—which also means the Pod network is **wide open by default** unless restricted by `NetworkPolicy`.

## Pod Lifecycle Phases

1. **`Pending`**: accepted by API server, waiting for scheduling or container image pull.
2. **`Running`**: bound to a node and at least one container is running or restarting.
3. **`Succeeded`**: all containers exited cleanly (`0`) and will not restart (batch `Job` Pods).
4. **`Failed`**: at least one container terminated with non-zero exit code.
5. **`Unknown`**: node communication lost (`kubelet` stopped heartbeating).

## QoS Classes & Eviction Order

Kubernetes assigns every Pod a **Quality of Service (QoS)** class based on container `requests` and `limits`:
- **`Guaranteed`**: every container sets `requests == limits` for both CPU and memory (evicted *last* under node memory pressure).
- **`Burstable`**: at least one container sets a request or limit, but not equal for all resources.
- **`BestEffort`**: no CPU/memory requests or limits set at all (evicted *first*).

## Unmanaged (Naked) Pods vs Static Pods vs Controllers

- **Naked/unmanaged Pod**: created directly via `kubectl apply` with a `kind: Pod` manifest—if it dies or its node fails, **nothing recreates it**. Always wrap workloads in a controller ([[K8S Deployment]], StatefulSet, DaemonSet, Job).
- **Static Pod**: managed directly by `kubelet` from local files on a node (e.g., `/etc/kubernetes/manifests`), bypassing the API server—used to bootstrap [[K8S Control Plane]] components (`kube-apiserver`, [[etcd]]).

> [!warning] Failure modes & trade-offs
> - **OOMKilled (exit 137) vs CPU throttling**: exceeding memory `limits` triggers the Linux kernel OOM killer immediately (`OOMKilled`), whereas exceeding CPU `limits` causes **CFS CPU throttling** (severe tail-latency spikes without crashing). Size memory limits with headroom (see [[JVM App Deployment]]) and be cautious with tight CPU limits on latency-sensitive services.
> - **`CrashLoopBackOff`**: not an error itself—it means the container repeatedly crashes and `kubelet` is applying exponential restart backoff (10s -> 20s -> ... -> 5m). Check `kubectl logs --previous` and `kubectl describe pod` for exit code (`137` OOM, `143` SIGTERM, `1` app crash).

## Flashcards

What resources do containers inside the same [[Kubernetes]] [[Pod]] share?::The same ==network namespace== (IP, ports, `localhost`), ==IPC namespace==, and mounted ==volumes== (all scheduled atomically on the same worker node).
^q-twvd
^q-k34y

What are the three Kubernetes Pod QoS classes in order from *last* evicted to *first* evicted under memory pressure?::1. ==`Guaranteed`== (`requests == limits`), 2. ==`Burstable`==, 3. ==`BestEffort`== (no requests/limits).
^q-hxta
^q-qtgj

### What is the difference between exceeding a Pod's memory limit vs exceeding its CPU limit? #card

Memory is incompressible: exceeding a container's memory `limit` causes the Linux kernel to immediately terminate the process (**`OOMKilled`**, exit code `137`). CPU is compressible: exceeding a CPU `limit` does not kill the Pod, but triggers **CFS CPU throttling**, which can cause severe p99 latency spikes.
^q-84yu

## Related
- [[Multi-container Pods]]
- [[K8S Deployment]]
- [[K8S ReplicaSet]]
- [[K8S Service]]
- [[K8S Core Containers]]
- [[JVM App Deployment]]
- [[Health Checks]]
