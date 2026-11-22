#add-cards

The process of moving load from one node in the cluster to another is called ::: rebalancing.

## Relation to Service Discovery

Rebalancing is also highly related to the topic of [[Service discovery]] as rebalancing changes the target IPs for user requests. Many companies have written their own in-house service discovery tools, and many of these have been released as open source.

On a high level there are several different approaches to this problem:
1. Load balance the request to any node, and node forwards the request to the correct target if not matched initially
2. Add a gateway that owns partitioning state
3. Let clients a way to learn the destination IP upfront.

The main problem is how the component making the routing decision learns the system state change 

## Reasons for rebalance #card

- The query throughput increases, so we have to add more CPUs
- The dataset size increases, so we need to add more disks and RAM to store it.
- A machine fails, and other machines need to take over the failed machine's responsibilities.


## Rebalancing minimum requirements #card 

No matter which partitioning scheme is used, rebalancing is usually expected to meet some minimum requirements:
- After rebalancing, the load (data storage and write requests) should be shared fairly between the nodes in the cluster.
- While rebalancing is happening, the database should ocntinue accepting reads and writes.
- No more data than necessary should be moved between nodes, to make rebalancing fast and to minimize the network and disk I/O load.

## Renalancing Strategies

### Fixed Number of Partitions #card

There is a simple solution: assign many more partitions than there are nodes. Each node receives lots of partitions. E.g. for cluster of 10 nodes we can assign 1000 paritions, and distribute them by 100 per node. When a node is added, it simply steals some partitions from other nodes to normalize the load.

This approach to rebalancing is used in [[Riak]], [[Elasticsearch]], [[Couchbase]], and [[Voldemort]]

In this configuration, the number of partitions is usually fixed when the database is first set up and not changed afterward. Technically the number of partitions could be changed, but not necessary.

Cons: hard too chose the right size - choose too big number of partitions, and this adds operational overhead. Choose too small - and there is too much overhead. Sarting with a low number of nodes will also require massive data transfer when the dataset grows and new nodes are added.

### Dynamic Partitioning

For databases that use key range partitioning, a fixed number of partitions with fixed boundaries would be very inconvenient: if you got the boundaries wrong, you could end up with all of the data in one partition and all of the other partitions empty. Reconfiguring the partition bound‐ aries manually would be very tedious.

For that reason, [[Data partitioning#Skewed Partition & Key-Range Partitioning|key range–partitioned]] databases such as HBase and RethinkDB create partitions dynamically. When a partition grows to exceed a configured size (on HBase, the default is 10 GB), it is split into two partitions so that approximately half of the data ends up on each side of the split. Conversely, if lots of data is deleted and a partition shrinks below some threshold, it can be merged with an adjacent partition. This process is similar to what happens at the top level of a B-tree.

Each partition is assigned to one node, and each node can handle multiple partitions, like in the case of a fixed number of partitions. After a large partition has been split, one of its two halves can be transferred to another node in order to balance the load. In the case of HBase, the transfer of partition files happens through [[HDFS]], the underlying distributed filesystem.

- Advantage: the number of partitions adapts to the total data volume (low for small, large for big).
- Downside: while the dataset is smal, initial, only one node is active. To mitigate this issue, [[HBase]] and [[MongoDB]] allow an initial set of partitions to be configured on an empty database (this is called *pre-splitting*). In the case of key-range partitioning, pre-splitting requires that you already know what the key distribution is going to look like.

### Partitioning Proportionally to nodes

The number of partitions proportional to the number of nodes—in other words, to have a fixed number of partitions per node. In this case, the size of each partition grows proportionally to the dataset size while the number of nodes remains unchanged, but when you increase the number of nodes, the partitions become smaller again. Since a larger data volume generally requires a larger number of nodes to store, this approach also keeps the size of each partition fairly stable.

Idea is to take some partitions from some nodes and halve them, and constuct new partitions from these halves. The randomization can produce unfair splits, but when averaged over a larger number of partitions (in Cassandra, 256 partitions per node by default), the new node ends up taking a fair share of the load from the existing nodes. Later Cassandra version 3.0+ introduces an alternative algorithm that avoids unfair split.

### How not to do it: hash mod N

If we assign partitions by simply modding the hash, each N change would cause massive aprtition change for data (e.g. 123 mod 10 != 123 mod 9 != 123 mod 11). This violates the rule that data shouldn't move more than necessary.

## Automatic or Manual Rebalancing

For example, Couchbase, Riak, and Voldemort generate a suggested partition assignment automatically, but require an administrator to commit it before it takes effect.


Automatic:
- Pros: less operational work to do the maintanence
- Cons: rebalancing is expensive. Automatic decision is unpredictable. Wrong rebalancing could overload the netweok and harm performance.

Such automation can be dangerous in combination with automatic failure detection. For example, say one node is overloaded and is temporarily slow to respond to requests. The other nodes conclude that the overloaded node is dead, and automatically rebalance the cluster to move load away from it. This puts additional load on the overloaded node, other nodes, and the network—making the situation worse and potentially causing a cascading failure.

## Parallel Query Execution

Massively parallel processing (MPP) relational database products, often used for analytics, are much more sophisticated in the types of queries they support. A typical data warehouse query contains several join, filtering, grouping, and aggregation operations. The MPP query optimizer breaks this complex query into a number of execution stages and partitions, many of which can be executed in parallel on different nodes of the database cluster. Queries that involve scanning over large parts of the dataset particularly benefit from such parallel execution.