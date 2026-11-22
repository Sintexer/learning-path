---
flashcards:
  q-gfgs: { nid: 1795361722281, hash: nn7sprrq, sync: bk8wekcz }
  q-f427: { nid: 1795361722304, hash: r6xxct7w, sync: fzbrxb2f }
  q-yjds: { nid: 1795361722329, hash: 7rx5kkpt, sync: vrkb9kgg }
  q-36rm: { nid: 1795361722354, hash: 2d7ebm88, sync: senj2aub }
  q-f2pc: { nid: 1795361722379, hash: 73f43fe5, sync: tnc3ikpn }
  q-jp5y: { nid: 1795361722403, hash: b5k7hb8m, sync: 6quwn7qu }
---

The process of moving load (data and request traffic) from one node in a cluster to another ::: rebalancing.
^q-gfgs

## Relation to Service Discovery & Request Routing

Rebalancing is closely tied to [[Service discovery]] because moving partitions changes which node IP owns a given key.

At a high level, there are three routing approaches:
1. **Node-to-node forwarding**: Route the request to any node via [[Load Balancing]]; if it does not own the partition, it forwards the request to the right node.
2. **Routing tier / gateway**: Send all requests through a partition-aware routing tier (similar to [[API Gateway]]) that forwards each request to the owning node.
3. **Partition-aware client**: Clients learn the partition-to-node mapping directly and connect to the target node without an extra hop.

The core challenge across all three approaches is keeping the routing component updated when partition ownership changes — typically solved with a consensus coordination service such as ZooKeeper or [[etcd]], or via gossip protocols (used in [[Apache Cassandra]] and Riak).

## Reasons for rebalance #card

- Query throughput increases, so we add more CPUs.
- Dataset size increases, so we add more disks and RAM.
- A machine fails, so surviving nodes must take over its partitions.
^q-f427

## Rebalancing minimum requirements #card

No matter which partitioning scheme is used, rebalancing is expected to meet three minimum requirements:
- After rebalancing, load (data storage, reads, and writes) should be shared fairly across nodes.
- While rebalancing is happening, the database must continue accepting reads and writes.
- No more data than necessary should move between nodes, keeping rebalancing fast and minimizing network and disk I/O overhead.
^q-yjds

## Rebalancing Strategies

### Fixed Number of Partitions #card

Create many more partitions than nodes (for example, 1,000 partitions across 10 nodes = 100 partitions per node). When a new node joins, it steals a few whole partitions from existing nodes until load is balanced.

Used in Riak, Elasticsearch, Couchbase, and Voldemort. Partition count is normally fixed at cluster creation.

- **Trade-off**: Choosing the partition count is hard — too many partitions add management/metadata overhead per partition; too few create oversized partitions that are slow and expensive to move or recover when the dataset grows.
^q-36rm

### Dynamic Partitioning

For [[Key-Range Partitioning|key-range partitioned]] databases (such as HBase and RethinkDB), fixed boundaries are impractical: bad initial boundaries leave all data in one partition and the rest empty.

Instead, partitions split dynamically when they exceed a size threshold (e.g., 10 GB in HBase) into two roughly equal halves, and merge with adjacent partitions if deletes shrink them — similar to the top level of a [[B-Tree Index|B-Tree]]. After a split, one half can be transferred to another node (in HBase, via HDFS).

- **Advantage**: Partition count adapts automatically to total data volume.
- **Downside**: An empty database starts with a single partition on one node until it reaches the split threshold. HBase and [[MongoDB]] mitigate this with **pre-splitting** (configuring initial boundaries upfront, which requires knowing the key distribution in advance).

### Partitioning Proportionally to Nodes

Maintain a fixed number of partitions *per node* (e.g., 256 vnodes per node in [[Apache Cassandra]]; see [[Consistent Hashing]]). Partition size grows with dataset size while node count is fixed, and shrinks when new nodes join.

When a new node joins, it randomly selects existing partitions to split in half, taking one half of each. Randomness can produce uneven splits at small counts, but averages out over many partitions per node (Cassandra 3.0+ uses an improved allocation algorithm to avoid skewed splits).

### How Not to Do It: `hash(key) % N`

Assigning partitions with `hash(key) % N` causes almost every key to move whenever node count `N` changes (`123 % 10 != 123 % 9 != 123 % 11`), violating the minimum-data-movement requirement.

## Automatic vs Manual Rebalancing

Systems like Couchbase, Riak, and Voldemort automatically compute a suggested partition assignment, but require an administrator to commit it before execution.

- **Fully automatic**: Less routine operational work, but rebalancing is expensive and unpredictable.
- **Semi-automatic (human-in-the-loop)**: Safer for large clusters because an operator confirms whether a node is truly dead before shifting terabytes across the network.

> [!warning] Failure modes & trade-offs
> - **Cascading overload from auto-rebalance + auto-failover**: If a node is merely overloaded or in a long GC pause, other nodes may declare it dead and start moving its partitions. The resulting network and disk storm overloads healthy nodes too, turning a single slow node into a cluster-wide outage.
> - **Stale routing tables during rebalance**: While ownership transfers, clients or routers with cached mappings can route writes to the old owner; systems must reject stale writes or forward them cleanly to avoid [[Split-Brain]] divergence.

## Parallel Query Execution

Massively parallel processing (MPP) analytical databases (see [[OLAP]] and [[Data Warehouse]]) go beyond key-value routing: the query optimizer breaks complex joins, filters, groupings, and aggregations into execution stages that run in parallel across partitions on many cluster nodes.

In dynamic key-range partitioning, an empty database starts with all writes hitting a single node until the first split; databases like HBase and MongoDB mitigate this with ==pre-splitting==.
^q-f2pc

Why is fully automatic rebalancing dangerous when combined with automatic failure detection?::A temporarily overloaded or slow node can be misclassified as dead, triggering heavy data transfers that saturate the network and healthy nodes — causing a cascading outage.
^q-jp5y

## Related
- [[Data partitioning]]
- [[Key-Range Partitioning]]
- [[Hash Partitioning]]
- [[Partitioning Hot Spots and Skew]]
- [[Partitioning and Secondary Indexes]]
- [[Consistent Hashing]]
- [[Service discovery]]
- [[Load Balancing]]
- [[etcd]]
- [[Apache Cassandra]]
- [[MongoDB]]
