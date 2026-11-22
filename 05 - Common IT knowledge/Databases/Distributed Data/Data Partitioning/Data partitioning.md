For very large datasets, or very high query throughput, [[Replication]] is not sufficient: we need to break the data up into **partitions**, also known as *sharding*.

> [!tip] Fact
> - Partition is called a **shard** in MongoDB, Elasticsearch, and SolrCloud; 
> - it’s known as a **region** in HBase, 
> - a **tablet** in Bigtable, 
> - a **vnode** in Cassandra and Riak,
> - and a **vBucket** in Couchbase. 
> 
> However, **partitioning** is the most established term.

The main reason for wanting to partition data is **scalability**. Different partitions can be placed on different nodes in a shared-nothing cluster. Thus, a large dataset can be distributed across many disks, and the query load can be distributed across many processors.

Partitioning is usually combined with [[Replication]] so that copies of each partition are stored on multiple nodes. This means that, even though each record belongs to exactly one partition, it may still be stored on several different nodes for fault tolerance.

For leader-based replication, different nodes are leaders and followers for different partitions.

## Skewed Partition

Our goal with partitioning is to spread the data and the query load evenly across nodes. If every node takes a fair share, then—in theory—10 nodes should be able to handle 10 times as much data and 10 times the read and write throughput of a single node (ignoring replication for now).

If the partitioning is unfair, so that some partitions have more data or queries than others, we call it **skewed**. The presence of skew makes partitioning much less effective. In an extreme case, all the load could end up on one partition, so 9 out of 10 nodes are idle and your bottleneck is the single busy node. A partition with disproportionately high load is called a **hot spot**.

Keeping partitions by sorting the key and using key ranges per partition might be a bad idea: yes, we can keep records sorted per partition, which increaes the range scans speed. However it inevitable skews data. E.g. for a timestamp key, data from sensors would load the same partition for the whole day.

To avoid this problem in the sensor database, you need to use something other than the timestamp as the first element of the key. For example, you could *prefix* each timestamp with the sensor name so that the partitioning is first by sensor name and then by time. Assuming you have many sensors active at the same time, the write load will end up more evenly spread across the partitions.

## Partitioning by Hash of Key

A good hash function takes skewed data and makes it uniformly distributed. 

> [!tip] Trivia
> For partitioning purposes, the hash function need not be cryptographically strong: for example, Cassandra and MongoDB use MD5, and Voldemort uses the *Fowler– Noll–Vo* function. Many programming languages have simple hash functions built in (as they are used for hash tables), but they may not be suitable for partitioning: for example, in Java’s Object.hashCode() and Ruby’s Object#hash, the same key may have a different hash value in different processes

Once you have a suitable hash function for keys, you can assign each partition a
range of hashes (rather than a range of keys), and every key whose hash falls within a
partition’s range will be stored in that partition. However now we lose the benefit of efficient range queries.

### Cassanra's solutions to the key hashing and range queries

Cassandra asked: _"How can we get the even distribution of the Hashing, WITHOUT losing the ability to keep related items sorted together on the same shelf?"_

The answer is the **Compound Primary Key**, which has two parts:  
`PRIMARY KEY (Partition_Key, Clustering_Key)`, usually it is something like `user_id` and `timestamp`

User id determines the partition, and DB uses the second part of the key (`timestamp`) to **physically sort records in chronological order right next to each other on the disk.**


## Skewed Workloads

Hashing can’t avoid skews entirely: in the extreme case where all reads and writes are for the same key, you still end up with all requests being routed to the same partition.

This kind of workload is perhaps unusual, but not unheard of: for example, on a social media site, a celebrity user with millions of followers may cause a storm of activity when they do something. This event can result in a large volume of writes to the same key (where the key is perhaps the user ID of the celebrity, or the ID of the action that people are commenting on). Hashing the key doesn’t help, as the hash of two identical IDs is still the same.

One way application solve this, is they append a random to-digits to the key - now the same key is distributed across 100 partitions. However it adds overhead on reading the data as it now has to be read from 100 key variants. It only makes sense to do this trick when the number of *hot spot* keys is small and apply it only to them.