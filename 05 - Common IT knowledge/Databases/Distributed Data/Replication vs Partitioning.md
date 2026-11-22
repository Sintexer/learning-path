[[Replication]] and [[Data partitioning]] are orthogonal, complementary techniques in [[Distributed Data|distributed databases]] — most large-scale systems combine both.

## [[Replication]]

Keeping copies of the **same data** on several different nodes, potentially across racks or geographic regions.
- **Primary goals**: Fault tolerance / [[Availability|high availability]], lower read latency (geo-locality), and read-throughput scaling.
- **Limitation**: Every replica still stores the entire dataset; does not solve dataset size or write throughput exceeding a single node.

## [[Data partitioning|Partitioning (Sharding)]]

Splitting a large dataset into **disjoint subsets** (partitions/shards) assigned to different nodes so each record belongs to exactly one partition.
- **Primary goals**: [[Horizontal scaling]] of storage capacity and write/query throughput beyond one machine.
- **Limitation**: Partitioning alone *reduces* availability (if 1 of 10 shards dies, 10% of data is unavailable) unless paired with replication.

## How They Combine in Production

In real clusters ([[Apache Cassandra]], [[Kafka]], [[MongoDB]], CockroachDB), the dataset is first **partitioned** into shards, and each shard is **replicated** across multiple nodes (typically replication factor $N = 3$). Each physical node acts as the leader for some partitions and a follower for others — balancing both storage and traffic evenly while surviving node failures.

| Dimension | [[Replication]] | [[Data partitioning\|Partitioning]] |
|---|---|---|
| **Data layout** | Same data copied to multiple nodes | Data split into disjoint subsets |
| **Scales** | Read throughput & fault tolerance | Storage size & write/read throughput |
| **Effect on availability** | Increases (redundancy) | Decreases without replication |
| **Key challenges** | [[Replication Lag]], failover, [[Split-Brain]], conflicts | Hot spots / skew, rebalancing, [[Partitioning and Secondary Indexes\|secondary indexes]] |

## How do Replication and Partitioning complement each other in a distributed database? #card
- **[[Data partitioning|Partitioning]]** divides data and write load across nodes for [[Horizontal scaling]], but makes each partition a single point of failure.
- **[[Replication]]** duplicates each partition across multiple nodes (e.g., $N = 3$) so any node can fail without losing data or availability.

Partitioning divides ==responsibility (each record belongs to one partition)==, while replication duplicates ==responsibility (multiple nodes hold copies of each partition)==.

## Related
- [[Replication]]
- [[Data partitioning]]
- [[Partitioning and Secondary Indexes]]
- [[Distributed Data]]
- [[Horizontal scaling]]
- [[Availability]]
