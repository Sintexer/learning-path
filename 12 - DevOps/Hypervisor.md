A **hypervisor** (Virtual Machine Monitor / VMM) is software, firmware, or hardware-assisted layer that virtualizes physical server resources (CPU, RAM, NICs, storage) so multiple isolated [[Virtual machine|virtual machines]] can run concurrently on a single physical host.

## Type 1 vs Type 2 Hypervisors

- **Type 1 (Bare-Metal)**: runs directly on physical hardware with no host OS underneath—high performance, low overhead, standard in data centers and public clouds.
  - *Examples*: **KVM** (Linux kernel module + QEMU/Firecracker, used by AWS Nitro & GCP), **VMware ESXi**, **Microsoft Hyper-V**, **Xen**.
- **Type 2 (Hosted)**: runs as an application on top of an existing host OS—convenient for local development, higher overhead.
  - *Examples*: VirtualBox, VMware Workstation/Fusion, Parallels.

## Hardware Virtualization & Cloud Evolution

Modern hypervisors rely on CPU extensions (Intel VT-x / AMD-V) so guest instructions execute directly on the CPU with minimal trapping. Modern clouds (like **AWS Nitro**) offload networking, storage, and hypervisor management onto dedicated hardware cards so nearly 100% of host CPU/RAM goes to customer VMs.

> [!warning] Failure modes & trade-offs
> - **Noisy neighbor & CPU steal (`%st`)**: on oversubscribed multi-tenant hosts, competing VMs can steal physical CPU cycles or saturate shared memory bandwidth/NICs—monitor `steal time` in Linux metrics.
> - **Live migration stalls**: migrating a VM across physical hosts can cause brief network/clock pauses that trigger sensitive [[Distributed Lock]] leases or consensus timeouts ([[Raft]]).

## Flashcards

Hypervisor:::Software or firmware layer (VMM) that virtualizes physical hardware so multiple isolated [[Virtual machine|virtual machines]] can share one host.

What is the difference between a Type 1 and a Type 2 hypervisor?::Type 1 (bare-metal: KVM, ESXi, Hyper-V) runs ==directly on physical hardware== for production/cloud; Type 2 (hosted: VirtualBox, VMware Fusion) runs ==on top of a host OS== for local dev.

High Linux ==CPU steal (`%st`)== inside a cloud VM indicates the hypervisor is throttling your vCPU because of noisy-neighbor contention on the underlying physical host.

## Related
- [[Virtual machine]]
- [[Difference between VMs and containers]]
- [[IaaS]]
- [[Container]]
