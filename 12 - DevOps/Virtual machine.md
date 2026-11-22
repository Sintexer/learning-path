A **virtual machine (VM)** emulates a physical server—complete with virtual CPU, memory, NIC, storage, and its own full guest operating system—allowing you to run isolated workloads as if on dedicated hardware. To run virtual machines, you install a [[Hypervisor|hypervisor]] on a physical host machine, which partitions and schedules hardware resources across VMs.

It is important to [[Difference between VMs and containers|differentiate VMs and containers]]. For application packaging and fast scaling, [[Container|containers]] are usually a better choice than bare VMs. In cloud environments ([[IaaS]]), containers typically run *inside* VMs—for example, an [[EC2|EC2]] instance acting as a [[K8S Worker nodes|Kubernetes worker node]] hosting dozens of containers simultaneously.

> [!tip] Production reality
> VMs remain the gold standard for **hard multi-tenant security isolation** (dedicated kernel per tenant) and for stateful infrastructure like self-managed databases or legacy OS workloads. Treat cloud VMs as immutable artifacts (baked with Packer and provisioned via [[Terraform]]) rather than long-lived "pet" servers patched manually over SSH.

## Flashcards

Virtual machine (VM):::A software-emulated computer running its own full guest OS and kernel on top of a [[Hypervisor]].

When are virtual machines still preferred over plain [[Container|containers]]?::When you need ==hardware/kernel-level isolation== (untrusted multi-tenancy), a different guest OS/kernel than the host, or dedicated-kernel predictability for stateful workloads.

## Related
- [[Hypervisor]]
- [[Difference between VMs and containers]]
- [[Container]]
- [[IaaS]]
- [[Terraform]]
