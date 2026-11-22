There are various reasons why you might want to distribute a database across multiple machines:
- **[[Scaling|Scalability]]**: If your data volume, read load, or write load grows bigger than a single machine can handle, you can spread the load across multiple machines (see [[Data partitioning]]).
- **Fault tolerance / [[Availability|high availability]]**: If your application needs to continue working even if one machine or datacenter goes down (see [[Replication]]).
- **Latency**: To locate databases geographically closer to users.

## Shared Memory Architecture

If all you need is to scale to higher load, the simplest approach is to buy a more powerful machine (sometimes called [[Vertical scaling|vertical scaling or scaling up]]). Many CPUs, many RAM chips, and many disks can be joined together under one operating system, and a fast interconnect allows any CPU to access any part of the memory or disk. In this kind of **shared-memory** architecture, all the components can be treated as a single machine.

The problem with a shared-memory approach is that the cost grows faster than linearly: a machine with twice as many CPUs, twice as much RAM, and twice as much disk capacity typically costs significantly more than twice as much. And due to bottlenecks, a machine twice the size cannot necessarily handle twice the load. A shared-memory architecture may offer limited fault tolerance — high-end machines have hot-swappable components — but it is strictly limited to a single geographic location.

## Shared Disk Architecture

Another approach is the **shared-disk** architecture, which uses several machines with independent CPUs and RAM, but stores data on an array of disks (SAN/NAS or cloud object/distributed storage like AWS Aurora) shared between the machines over a fast network. This architecture is used for some [[Data Warehouse|data warehousing]] workloads, though lock contention and network I/O overhead can limit write scalability.

## Shared Nothing Architecture

**Shared-nothing** architectures (sometimes called [[Horizontal scaling|horizontal scaling or scaling out]]) have gained the most popularity for distributed databases. In this approach, each machine or [[Virtual machine|virtual machine]] running the database software is called a **node**. Each node uses its CPUs, RAM, and disks independently. Any coordination between nodes is done at the software level, using a conventional network.

No special hardware is required by a shared-nothing system, so you can use commodity machines with the best price/performance ratio. You can distribute data across multiple geographic regions, reduce latency for users, and survive the loss of an entire datacenter.

> [!warning] Failure modes & trade-offs
> Distributing data across a shared-nothing cluster introduces fundamental problems that do not exist on a single node: partial failures, unreliable networks and clocks, [[Replication Lag]], [[Split-Brain]], and expensive cross-node coordination ([[CAP Theorem]], [[Consistency Levels]]). Never distribute a database before exhausting [[Vertical scaling]] and read replicas — a single modern NVMe server handles far more load than most teams expect.

## Compare Shared-Memory, Shared-Disk, and Shared-Nothing database architectures. #card
- **Shared-Memory ([[Vertical scaling]])**: Single OS with many CPUs/RAM/disks. Simple, no distributed semantics, but super-linear cost and single-region failure domain.
- **Shared-Disk**: Independent compute nodes sharing a storage array. Common in [[Data Warehouse|data warehouses]] and cloud-native DBs (Aurora), but lock/network contention can bottleneck writes.
- **Shared-Nothing ([[Horizontal scaling]])**: Independent nodes (CPU + RAM + disk) coordinating over the network. Best price/performance and multi-region fault tolerance, but adds full distributed-systems complexity.

Do not shard a database prematurely: exhaust [[Vertical scaling]], indexing, and read replicas before paying the operational tax of [[Data partitioning]] and cross-node transactions. #card-reminder

## Related
- [[Replication]]
- [[Data partitioning]]
- [[Replication vs Partitioning]]
- [[Consistency Levels]]
- [[CAP Theorem]]
- [[Horizontal scaling]]
- [[Vertical scaling]]
- [[Designing Data Intensive Applications]]
