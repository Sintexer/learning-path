To build a robust [[Zero-downtime deployment]] for a JVM application, you need to wire JVM bootstrap, memory sizing, and shutdown semantics cleanly into the [[Kubernetes]] and [[Container]] lifecycle.

## Startup, Readiness, and Liveness Probes ([[Health Checks]])

JVM apps take time to warm up the JIT, load the Spring context, and initialize DB connection pools. If [[Kubernetes]] routes traffic to a booting [[Pod]], clients hit connection errors and slow responses.

- **Startup Probe**: protects slow-starting JVM containers so the liveness probe doesn't prematurely kill the pod during Spring boot.
- **Readiness Probe** (`/actuator/health/readiness`): answers *"Is this Pod ready to accept HTTP traffic?"* Kubernetes keeps the pod out of [[K8S Service]] `EndpointSlices` until it returns `200 OK`.
- **Liveness Probe** (`/actuator/health/liveness`): answers *"Is this process stuck in a deadlock/broken state and needs a restart?"* Never include external dependencies (DB, [[Kafka]], downstream APIs) in liveness checks—otherwise a DB hiccup causes a cluster-wide restart storm.

## Graceful Shutdown & the `preStop` / `SIGTERM` Race

During a rolling update or scale-in, Kubernetes marks the [[Pod]] `Terminating` and executes **two actions asynchronously in parallel**:
1. The EndpointSlice controller removes the Pod IP from `EndpointSlices`, and `kube-proxy` / Ingress controllers asynchronously update routing rules across all nodes (can take 1–5+ seconds).
2. `kubelet` runs the `preStop` hook (if configured), then sends `SIGTERM` to the container's PID 1.

If Spring receives `SIGTERM` immediately—even with `server.shutdown=graceful`—it stops accepting *new* connections while `kube-proxy` / Ingress is **still routing new requests** to the pod, causing `502`/`503` errors!

**Production pattern:**
1. Set `server.shutdown: graceful` and `spring.lifecycle.timeout-per-shutdown-phase: 30s` in `application.yaml`.
2. Add a `preStop` sleep (`sleep 5` to `sleep 10`) so the pod keeps serving in-flight and late-arriving requests while `EndpointSlices` / `kube-proxy` / cloud LBs finish draining it.
3. Ensure `terminationGracePeriodSeconds` > (`preStop` sleep + `timeout-per-shutdown-phase`) so `kubelet` doesn't send `SIGKILL` mid-drain.
4. Use exec-form `ENTRYPOINT ["java", ...]` (not shell form) so the JVM is PID 1 and actually receives `SIGTERM`.

## Container Memory & CPU Sizing

- Modern JVMs (Java 11+) are container-aware (`-XX:+UseContainerSupport`, on by default), respecting cgroup memory/CPU limits. Set `-XX:MaxRAMPercentage=75.0` (leaving headroom for Metaspace, thread stacks, direct buffers, and native memory) rather than hardcoding `-Xmx` equal to the container memory limit—otherwise the Linux OOM killer terminates the pod (`OOMKilled`, exit code 137) before GC runs.
- Watch out for **CPU throttling**: setting tight Kubernetes CPU `limits` can throttle GC/JIT bursts and spike p99 latency even when average CPU looks low. Many teams set `requests == limits` for memory (Guaranteed QoS for memory) while leaving CPU limits unset or generous.

> [!warning] Failure modes & trade-offs
> - **OOMKilled (exit 137) vs `OutOfMemoryError`**: if `Xmx` + off-heap (Metaspace, Netty direct memory, JNI) exceeds the pod's cgroup memory limit, the kernel kills the JVM instantly with no heap dump. Always reserve 20–30% off-heap headroom.
> - **Dependencies in liveness probe**: checking [[PostgreSQL]] or [[Redis]] inside `/actuator/health/liveness` turns a brief DB blip into a cascading restart storm of every JVM pod (making recovery much slower due to Spring boot + JIT warmup).

## Flashcards

Why does `server.shutdown=graceful` alone still drop requests during a [[Kubernetes]] rolling update?::Because `EndpointSlice`/`kube-proxy` removal and `SIGTERM` delivery happen ==in parallel==; a `preStop` sleep (e.g., 5–10s) delays `SIGTERM` until load balancers stop routing new traffic to the pod.

In Kubernetes, `terminationGracePeriodSeconds` must be strictly greater than ==`preStop` duration + application graceful-shutdown timeout== (otherwise `kubelet` sends `SIGKILL` mid-drain).

Why should you never check external dependencies (DB, [[Redis]], downstream APIs) inside a Kubernetes liveness probe?::A brief downstream outage would fail liveness on all pods simultaneously, triggering a cluster-wide ==restart storm== that worsens the outage.

### Why can a JVM container get `OOMKilled` (exit 137) even when heap usage is below `-Xmx`? #card

Because the Kubernetes cgroup memory limit applies to **total process RSS** (Java heap + Metaspace + code cache + thread stacks + direct byte buffers + native allocations). Setting `-Xmx` near 100% of the container limit leaves no room for off-heap memory, causing the Linux kernel OOM killer to terminate the pod.

## Related
- [[Zero-downtime deployment]]
- [[Health Checks]]
- [[Kubernetes]]
- [[K8S Deployment]]
- [[Pod]]
- [[jvmArgs]]
- [[Container]]
