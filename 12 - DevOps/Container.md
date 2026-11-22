---
flashcards:
  q-i397: { nid: 1795361686230, hash: 2bj4vsh8, sync: iv9ebn3a }
  q-c97q: { nid: 1795361686303, hash: e3gx5eqf, sync: kfdzs6xy }
  q-qm3d: { nid: 1795356716817, hash: zxg6kaf9, sync: awpp9idp }
---

**What is a container?** Imagine a container as a lightweight, isolated environment that encapsulates an **application** and **all its dependencies** (libraries, configuration files, etc.). It's like a **self-contained package** that allows the application to *run consistently across different environments*, regardless of the underlying infrastructure. It is important to [[Difference between VMs and containers|differentiate VMs and containers]]. Containers share the *same operating system and kernel* as the host that they exist on, whereas [[Virtual machine|virtual machines]] contain their *own operating system*.

**Key characteristics:**

* **Isolation:** Each container runs in its own isolated user space, preventing conflicts with other applications or the host system.
* **Portability:** Standardized by the [[Open Container Initiative]] (OCI), container images run identically across dev laptops, CI, and production clusters.
* **Lightweight:** Compared to [[Virtual machine|virtual machines]], containers have a much smaller footprint and start in milliseconds to seconds.
* **Scalability:** Containers are easy to scale out ([[Horizontal scaling]]) to meet changing demand.

In the real world it is hard to manage containers manually. Various [[Containers orchestration|orchestration services]] like [[AWS Elastic Container Service|Amazon ECS]] or [[Kubernetes|K8S]] manage hundreds or thousands of containers simultaneously, spin up and shut down instances on demand, and provide [[Service discovery|service discovery]].

### Use cases for containers

* **[[Microservices architecture]]:** Ideal for packaging and deploying small, independently releasable services.
* **Continuous integration/continuous delivery (CI/CD):** Provides reproducible build, test, and runtime environments.
* **[[Cloud-Native App|Cloud-native applications]]:** Enables immutable deployments, rolling updates, and self-healing.

### How do containers work under the hood?

Containers are **not** mini-VMs—they are ordinary Linux processes isolated by kernel primitives:
- **Linux Namespaces** (*what you can see*): isolate PID tree, network stack (`net`), filesystem mounts (`mnt`), UTS hostname, IPC, and UID/GID (`user`).
- **cgroups (v1/v2)** (*what you can use*): enforce CPU, memory, PID, and I/O quotas/limits.
- **Union filesystem (OverlayFS)**: stacks read-only image layers with a thin writable container layer (copy-on-write).
- **Security primitives**: Linux capabilities, `seccomp`, and AppArmor/SELinux restrict what syscalls the process can invoke on the shared kernel.

> [!warning] Failure modes & trade-offs
> - **Shared kernel blast radius**: a kernel vulnerability or container escape compromises the host and neighbor containers—never run untrusted multi-tenant code in plain containers without sandboxing (gVisor, Kata Containers, or Firecracker microVMs).
> - **Running as `root` (UID 0)**: by default many images run as root inside the container; always set a non-root `USER`, drop Linux capabilities, and use a read-only root filesystem where possible.
> - **PID 1 zombie & signal trap**: if your app runs as PID 1 without signal handling (or via `sh -c`), `SIGTERM` may be ignored until `SIGKILL`, breaking [[Zero-downtime deployment|graceful shutdown]].

## Flashcards

What two Linux kernel primitives provide the core isolation behind containers?::==Namespaces== (isolate what a process can see: PID, net, mount, user) and ==cgroups== (limit what resources a process can use: CPU, memory, I/O).
^q-hg6f
^q-i397

Why are standard containers weaker security boundaries than [[Virtual machine|VMs]] for untrusted multi-tenant workloads?::Because containers share the ==host OS kernel==; a kernel exploit can break out to the host, so untrusted multi-tenant workloads use microVMs (Firecracker) or sandboxed runtimes (gVisor/Kata).
^q-f2jq
^q-c97q

Always run production containers as a non-root user with a read-only root filesystem and minimal Linux capabilities. #card-reminder
^q-qm3d

## Related
- [[Difference between VMs and containers]]
- [[Open Container Initiative]]
- [[Containers orchestration]]
- [[Container Runtime Interface]]
- [[containerd]]
- [[Kubernetes]]
- [[Virtual machine]]
