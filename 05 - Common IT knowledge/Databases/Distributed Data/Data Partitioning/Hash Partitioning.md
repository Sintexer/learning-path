---
flashcards:
  q-dpct: { nid: 1795361722178, hash: wuhg3bnb, sync: 3vvfvdcb }
  q-q4t6: { nid: 1795361722205, hash: yfmethpn, sync: 543kvaea }
---

In **hash partitioning**, a hash function maps each key to a uniformly distributed number, and each partition owns a range of hash values rather than a range of raw keys. Even closely related keys (e.g., `user_1`, `user_2`, `user_3`) scatter evenly across partitions, avoiding the monotonic-key skew of [[Key-Range Partitioning]].

> [!tip] Trivia
> The hash function does not need to be cryptographically strong: [[Apache Cassandra]] and [[MongoDB]] use MD5, and Voldemort uses *Fowler–Noll–Vo (FNV)*. Built-in language hashes such as Java's `Object.hashCode()` or Ruby's `Object#hash` are **unsuitable** because the same key can hash to different values in different processes.

## Trade-Off: Losing Range Queries

Because adjacent keys are scattered across different partitions, **efficient range queries across primary keys are lost**. A range scan must be broadcast to every partition (scatter/gather), similar to a local index scan in [[Partitioning and Secondary Indexes]].

## Cassandra's Compound Primary Key

[[Apache Cassandra]] combines uniform distribution with localized range scans using a **Compound Primary Key**:

```sql
PRIMARY KEY (partition_key, clustering_key) -- e.g. (user_id, timestamp)
```

- `partition_key` (`user_id`) is **hashed** to pick the owning partition evenly across the cluster.
- `clustering_key` (`timestamp`) **physically sorts** rows within that partition on disk (inside an [[SSTable]]), enabling fast chronological range queries for a single user without scanning other nodes.

> [!warning] Failure modes & trade-offs
> - **Unbounded partition growth**: In Cassandra's compound key model, if one `partition_key` accumulates millions of `clustering_key` rows (e.g., an active IoT device or celebrity feed), it creates a **wide partition** that causes GC pauses, slow compactions, and tombstones. Bucket the partition key (e.g., `(user_id, yyyy_mm)`).
> - **Same-key hot spots remain**: Hashing distributes *distinct* keys evenly, but it cannot split traffic targeting a single identical key (see [[Partitioning Hot Spots and Skew]]).

Why are language built-in hash functions like Java's `Object.hashCode()` unsuitable for database hash partitioning?::Their output is not guaranteed to be consistent across different processes or JVM restarts, so nodes would route the same key to different partitions.
^q-dpct

Hash partitioning achieves uniform key distribution across nodes at the cost of ==losing efficient range queries across primary keys==.
^q-q4t6

## Related
- [[Data partitioning]]
- [[Key-Range Partitioning]]
- [[Partitioning Hot Spots and Skew]]
- [[Consistent Hashing]]
- [[Rebalancing Partitions]]
- [[Apache Cassandra]]
- [[MongoDB]]
