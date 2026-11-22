---
flashcards:
  q-5b4a: { nid: 1795356725143, hash: n25nnx8d, sync: 5vfa6rzr }
  q-vvcn: { nid: 1795356725170, hash: 6n6975ar, sync: 4wvaint9 }
---

For very large datasets, or very high query throughput, [[Replication]] is not sufficient: we need to break the data up into **partitions**, also known as *sharding* (see [[Replication vs Partitioning]]).

> [!tip] Fact
> - Partition is called a **shard** in [[MongoDB]], Elasticsearch, and SolrCloud;
> - it’s known as a **region** in HBase,
> - a **tablet** in Bigtable,
> - a **vnode** in [[Apache Cassandra]] and Riak,
> - and a **vBucket** in Couchbase.
>
> However, **partitioning** is the most established term.

The main reason for wanting to partition data is **[[Horizontal scaling|scalability]]**. Different partitions can be placed on different nodes in a [[Distributed Data#Shared Nothing Architecture|shared-nothing cluster]]. Thus, a large dataset can be distributed across many disks, and the query load can be distributed across many processors.

Partitioning is usually combined with [[Replication]] so that copies of each partition are stored on multiple nodes. Even though each record belongs to exactly one partition, it is stored on several nodes for fault tolerance. For [[Leader-Based Replication|leader-based replication]], different nodes are leaders and followers for different partitions.

## Partitioning Building Blocks

- **[[Key-Range Partitioning]]** — partitions own sorted key ranges (fast range scans, vulnerable to monotonic-key write skew).
- **[[Hash Partitioning]]** — partitions own hash ranges (uniform distribution, loses range scans; mitigated by Cassandra's compound primary key).
- **[[Partitioning Hot Spots and Skew]]** — handling uneven load and celebrity keys via key salting.
- **[[Partitioning and Secondary Indexes]]** — local (document-partitioned, scatter/gather) vs global (term-partitioned, async) secondary indexes.
- **[[Rebalancing Partitions]]** — moving partitions safely when cluster membership changes (fixed partitions, dynamic splitting, proportional vnodes) plus request routing.

> [!warning] Failure modes & trade-offs
> - **Automatic rebalancing cascades**: Automated rebalancing combined with aggressive failure detection can mistake a slow/GC-paused node for dead, triggering massive data transfers that overload the network and healthy nodes — causing a cascading outage. Keep a human in the loop for rebalance approval in large clusters.
> - **Cross-partition transactions**: Once data is partitioned, multi-record updates spanning partitions lose cheap local [[ACID]] transactions and require [[Two-Phase Commit]] or [[Saga]]s.

## Why is `hash(key) % N` an anti-pattern for partitioning a distributed database? #card
Whenever the number of nodes `N` changes (adding capacity or replacing a failed node), `hash(key) % N` changes the target node for almost every key in the database, triggering massive, expensive data movement across the entire cluster. Use fixed-count virtual partitions, dynamic splitting, or [[Consistent Hashing]] instead.
^q-5b4a

## How does Cassandra's compound primary key combine hash partitioning with range queries? #card
`PRIMARY KEY (partition_key, clustering_key)` hashes only the `partition_key` to choose the partition evenly, then sorts all rows within that partition by `clustering_key` — enabling efficient range scans within a single partition key (e.g., one user's timeline by timestamp).

To mitigate a single **hot key** (celebrity problem) under hash partitioning, applications can ==salt the key with a random suffix (e.g., `00`–`99`)==, trading extra read fan-out for distributed write load.
^q-vvcn

## Related
- [[Key-Range Partitioning]]
- [[Hash Partitioning]]
- [[Partitioning Hot Spots and Skew]]
- [[Partitioning and Secondary Indexes]]
- [[Rebalancing Partitions]]
- [[Replication vs Partitioning]]
- [[Consistent Hashing]]
- [[Replication]]
- [[Apache Cassandra]]
- [[MongoDB]]
- [[Kafka Partition]]
- [[Distributed Data]]
