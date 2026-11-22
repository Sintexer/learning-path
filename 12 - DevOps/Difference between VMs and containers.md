---
flashcards:
  q-ed6k: { nid: 1795361685203, hash: yfffr79x, sync: i3cdhr5f }
  q-uete: { nid: 1795356716641, hash: cbbbawky, sync: 68e64ym5 }
---

Compared to [[Virtual machine|virtual machines (VMs)]], [[Container|containers]] share the *same operating system and kernel* as the host they are deployed on.

**Containers** share the host OS kernel and isolate processes using Linux namespaces and cgroups. **Virtual machines** run a full guest OS and dedicated kernel on virtualized hardware managed by a [[Hypervisor]]. Each VM carries the memory, disk, and patching overhead of its own OS.

A container is much more lightweight and **spins up in milliseconds to seconds**, which is crucial when [[Horizontal scaling|scaling out]] quickly during traffic bursts. VMs boot in tens of seconds to minutes, but provide stronger hardware-level isolation and let you run different OSes/kernels (e.g., Windows and Linux) on the same physical host.

| Dimension | [[Virtual machine\|Virtual Machine (VM)]] | [[Container]] |
| --- | --- | --- |
| **Isolation boundary** | Hardware virtualization via [[Hypervisor]] (dedicated kernel) | Host OS kernel primitives (namespaces + cgroups) |
| **Startup time** | Seconds to minutes | Milliseconds to seconds |
| **Overhead** | GBs of RAM/disk per guest OS | MBs; near-native CPU/memory efficiency |
| **Security isolation** | Strong (safe for untrusted multi-tenancy) | Weaker (shared kernel attack surface) |
| **Typical pairing** | Runs the cluster hosts ([[K8S Worker nodes]]) | Runs the application workloads inside those VMs |

In practice they are complementary: cloud workloads routinely run **containers inside VMs** (e.g., [[Kubernetes]] worker nodes running on EC2 instances), combining VM-level tenant/cluster isolation with container-level density and portability. MicroVMs (like AWS Firecracker, powering [[AWS Lambda]] and [[AWS Fargate]]) bridge both worlds with VM isolation and sub-second startup.

![[Difference between VMs and containers visual.png]]

## Flashcards

What is the fundamental architectural difference between a [[Virtual machine|VM]] and a [[Container]]?::A VM virtualizes hardware via a [[Hypervisor]] and runs its own ==guest OS and kernel==; a container shares the ==host OS kernel== and isolates processes via Linux namespaces and cgroups.
^q-9qnp
^q-ed6k

Why do cloud platforms commonly run [[Container|containers]] *inside* [[Virtual machine|VMs]]?::To combine VM-level hardware isolation (separating tenants/clusters) with container-level fast startup, packaging portability, and high bin-packing density.
^q-uete

## Related
- [[Container]]
- [[Virtual machine]]
- [[Hypervisor]]
- [[Kubernetes]]
- [[Serverless]]
