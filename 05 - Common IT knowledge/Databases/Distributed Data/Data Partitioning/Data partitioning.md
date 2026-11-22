---
flashcards:
  q-5b4a: { nid: 1795356725143, hash: v22faf43, sync: s8mnuyq4 }
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

Partitioning is usually combined with [[Replication]] so that copies of each partition are stored on multiple nodes. This means that, even though each record belongs to exactly one partition, it may still be stored on several different nodes for fault tolerance.

For [[Leader-Based Replication|leader-based replication]], different nodes are leaders and followers for different partitions.

## Skewed Partition & Key-Range Partitioning

Our goal with partitioning is to spread the data and the query load evenly across nodes. If every node takes a fair share, then—in theory—10 nodes should be able to handle 10 times as much data and 10 times the read and write throughput of a single node (ignoring replication for now).

If the partitioning is unfair, so that some partitions have more data or queries than others, we call it **skewed**. The presence of skew makes partitioning much less effective. In an extreme case, all the load could end up on one partition, so 9 out of 10 nodes are idle and your bottleneck is the single busy node. A partition with disproportionately high load is called a **hot spot**.

Keeping partitions by sorting the key and using **key ranges** per partition lets you keep records sorted per partition, which increases range scan speed. However, it can easily skew writes: e.g., for a timestamp key, data from sensors would load the exact same partition for the entire day.

To avoid this problem in a sensor database, you need to use something other than the timestamp as the first element of the key. For example, you could *prefix* each timestamp with the sensor name so that partitioning is first by sensor name and then by time. Assuming many sensors are active at the same time, the write load will be spread evenly across partitions.

## Partitioning by Hash of Key

A good hash function takes skewed data and makes it uniformly distributed.

> [!tip] Trivia
> For partitioning purposes, the hash function need not be cryptographically strong: for example, [[Apache Cassandra]] and [[MongoDB]] use MD5, and Voldemort uses the *Fowler–Noll–Vo* function. Many programming languages have simple hash functions built in (as they are used for hash tables), but they may not be suitable for partitioning: for example, in Java’s `Object.hashCode()` and Ruby’s `Object#hash`, the same key may have a different hash value in different processes.

Once you have a suitable hash function for keys, you can assign each partition a range of hashes (rather than a range of keys), and every key whose hash falls within a partition’s range will be stored in that partition (see [[Consistent Hashing]]). However, you lose efficient range queries across keys because adjacent keys are scattered across all partitions.

### Cassandra's solution to key hashing and range queries

[[Apache Cassandra]] asked: *"How can we get the even distribution of hashing, WITHOUT losing the ability to keep related items sorted together on the same shelf?"*

The answer is the **Compound Primary Key**, which has two parts:
`PRIMARY KEY (Partition_Key, Clustering_Key)` — usually something like `(user_id, timestamp)`.

`user_id` is hashed to determine the partition, and the DB uses the second part of the key (`timestamp`) to **physically sort records in chronological order right next to each other on disk** (within an [[SSTable]]), enabling fast range scans for a single user.

## Skewed Workloads (Hot Keys)

Hashing can’t avoid skew entirely: in the extreme case where all reads and writes are for the *same* key, all requests still route to the same partition.

This kind of workload happens on social media sites when a celebrity user with millions of followers causes a storm of activity on a single key (the celebrity's user ID or post ID). Hashing the key doesn’t help, because identical keys always produce the identical hash.

One application-level mitigation is **key salting**: append a random two-digit number (`00`–`99`) to the hot key so writes spread across 100 partitions. However, reads now have to query and merge all 100 key variants, so this trick only makes sense for a small, known set of hot-spot keys.

## Rebalancing & Request Routing

When nodes are added or removed, partitions must be **rebalanced** without moving unnecessary data:
- **Never use `hash(key) % N`**: changing `N` moves almost every key in the cluster.
- **Fixed number of partitions**: create far more partitions than nodes (e.g., 1,000 partitions for 10 nodes) and move whole partitions between nodes (used by Riak, Elasticsearch, Couchbase).
- **Dynamic partitioning**: split a partition in two when it exceeds a size threshold, merge when too small (HBase, RethinkDB).
- **[[Consistent Hashing]] / proportional partitions**: fixed number of partitions per node (Cassandra vnodes).

For secondary indexes across partitions, see [[Partitioning and Secondary Indexes]].

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
- [[Partitioning and Secondary Indexes]]
- [[Replication vs Partitioning]]
- [[Consistent Hashing]]
- [[Replication]]
- [[Apache Cassandra]]
- [[MongoDB]]
- [[Kafka Partition]]
- [[Distributed Data]]
