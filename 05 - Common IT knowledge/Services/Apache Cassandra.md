A distributed, wide-column [[NoSQL database]] built for high write throughput, linear [[Horizontal scaling]], and multi-datacenter [[Availability]] with no single point of failure.

## Architecture & Data Model

- **Ring topology & [[Consistent Hashing]]:** Data is partitioned across nodes using a partitioner (Murmur3) over the **partition key**. Any node can act as a coordinator for any request.
- **[[Leaderless Replication]]:** Inspired by Amazon Dynamo, writes and reads use tunable [[Consistency Levels]] (`ONE`, `QUORUM`, `LOCAL_QUORUM`, `ALL`) governed by quorum math ($W + R > N$ for strong overlap).
- **Storage engine:** Each node writes sequentially to a commit log ([[WAL]]) and an in-memory memtable, then flushes immutable [[SSTable|SSTables]] to disk using an [[LSM Tree]] architecture.
- **Query-driven modeling:** Tables are designed around access patterns (primary key = `(partition_key, clustering_columns)`). Joins, ad-hoc filters, and multi-partition [[Transaction|transactions]] are avoided.
- **Background convergence:** Relies on [[Read Repair]], hinted handoffs ([[Sloppy Quorum]]), and [[Anti-Entropy]] (Merkle tree repairs) to heal diverged replicas.

> [!warning] Failure modes & trade-offs
> - **Tombstone accumulation:** Deletes write markers called *tombstones* instead of removing data immediately. Heavy delete/TTL workloads cause read timeouts and GC pauses if queries scan thousands of tombstones before compaction clears them.
> - **Wide & hot partitions:** Unbounded partition keys (e.g., `sensor_id` without a time bucket) create multi-gigabyte partitions that trigger OOMs and hot nodes. Always bucket time-series keys (e.g., `(sensor_id, day)`).
> - **Lightweight Transactions (LWT):** Cassandra supports compare-and-set (`IF NOT EXISTS`) via [[Paxos]], which adds 4 round-trips and severely degrades throughput — use sparingly.

## Flashcards

In Cassandra, ==partition key== determines which nodes own a row, while ==clustering columns== determine the sort order on disk within that partition.

Why do heavy delete or TTL workloads degrade Cassandra read performance?::Deletes create tombstones that remain until compaction and `gc_grace_seconds` expire, forcing reads to scan over dead markers.

## What makes Cassandra achieve very high write throughput? #card

- [[Leaderless Replication]] lets any node coordinate writes without single-leader bottlenecks.
- Writes append sequentially to a commit log and in-memory memtable ([[LSM Tree]]), avoiding random disk I/O on the write path.

Design Cassandra tables around query patterns and bucket time-series partition keys to prevent unbounded wide partitions. #card-reminder

## Related

- [[Leaderless Replication]]
- [[LSM Tree]]
- [[SSTable]]
- [[Consistency Levels]]
- [[Sloppy Quorum]]
- [[Anti-Entropy]]
- [[Read Repair]]
- [[Consistent Hashing]]
- [[Paxos]]
