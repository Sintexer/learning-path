**Replication** means keeping a copy of the same data on multiple machines connected via a network (see [[Replication vs Partitioning]]). There are three main reasons to replicate data:
1. **Latency** — place data geographically closer to users.
2. **Fault tolerance / [[Availability|high availability]]** — keep the system working if some nodes or a datacenter fail.
3. **[[Horizontal scaling|Read throughput scaling]]** — scale out the number of machines serving read queries.

Each node that stores a copy of the database is called a **replica**. With multiple replicas, the central challenge is: *how do we ensure that all changes end up on all replicas reliably?*

Three dominant replication architectures exist:

1. **[[Leader-Based Replication]]** (*primary-replica* / *read-scaling architecture*): One leader accepts all writes and streams a [[Replication Log]] to followers. Reads can scale out across followers (realistically with asynchronous replication). An important choice here is whether replication happens [[Synchronous vs Asynchronous Replication|synchronously or asynchronously]]. When an application reads from an async follower, it may see outdated results due to [[Replication Lag]], though replicas eventually catch up — a phenomenon called [[Eventual Consistency]].
2. **[[Multi-Leader Replication]]**: Multiple leaders (typically one per datacenter) accept writes to lower geo-latency and survive WAN outages. However, the overhead and write-conflict complexity outweigh the benefits inside a single datacenter — use it only for multi-datacenter setups, offline clients, or collaborative editing.
3. **[[Leaderless Replication]]** (*Dynamo-style*, e.g., [[Apache Cassandra]], Riak): Abandons the concept of a leader and lets clients send reads and writes directly to multiple replicas in parallel, using [[Quorum Consensus|quorums]] ($w + r > n$), [[Read Repair]], and [[Anti-Entropy]] to stay consistent.

> [!tip] Production reality
> Replication topology (single-leader, multi-leader, leaderless) and consistency guarantees ([[Consistency Levels]]) are **independent design dimensions**. For example, single-leader replication with asynchronous followers is *not* linearizable on follower reads, while leaderless replication with strict quorums ($w + r > n$) still does not guarantee [[Linearizability]] without extra coordination.

## Compare the three main replication architectures and their primary trade-off. #card
- **[[Leader-Based Replication]]**: Simple, no write conflicts; bottlenecked by single write leader and failover/[[Replication Lag]] hazards.
- **[[Multi-Leader Replication]]**: Low multi-region write latency and offline support; introduces asynchronous **write conflicts** and complex topologies.
- **[[Leaderless Replication]]**: No failover needed, high write availability (especially with [[Sloppy Quorum]]); complex concurrent-write handling (LWW data loss / version vectors) and unbounded staleness without [[Anti-Entropy]].

What are the three primary reasons to replicate data across nodes?::1. Lower latency (geo-proximity), 2. High availability / fault tolerance, 3. Scaling read throughput horizontally.

## Related
- [[Leader-Based Replication]]
- [[Multi-Leader Replication]]
- [[Leaderless Replication]]
- [[Synchronous vs Asynchronous Replication]]
- [[Replication Log]]
- [[Replication Lag]]
- [[Replication vs Partitioning]]
- [[Eventual Consistency]]
- [[Consistency Levels]]
