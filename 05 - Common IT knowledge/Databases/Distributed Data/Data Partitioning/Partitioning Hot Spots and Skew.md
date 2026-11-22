---
flashcards:
  q-jyyu: { nid: 1795361722104, hash: h76kdw3e, sync: v8kh3v53 }
  q-yqdc: { nid: 1795361722129, hash: y3d2ifi9, sync: wcbtxtxu }
  q-qz37: { nid: 1795361722153, hash: fear2ssr, sync: piujr4ih }
---

The goal of [[Data partitioning]] is to spread data storage and query load evenly across nodes. When distribution is uneven so that some partitions carry far more data or traffic than others, the partitioning is **skewed**. A partition with disproportionately high load is a **hot spot**.

In the worst case, 9 of 10 nodes sit idle while 1 hot partition bottlenecks the entire cluster, defeating [[Horizontal scaling]].

## Common Causes of Hot Spots

1. **Monotonic keys under [[Key-Range Partitioning]]**: Timestamps or sequential IDs route all new inserts to the latest range partition.
2. **Celebrity / single-key workload under [[Hash Partitioning]]**: Even a perfect hash function hashes the same key (`celebrity_user_id`) to the exact same partition every time. Millions of concurrent reads or writes hammer a single node.
3. **Low-cardinality global secondary indexes**: In [[Partitioning and Secondary Indexes|term-partitioned secondary indexes]], indexing a field like `status = 'ACTIVE'` concentrates writes on one index partition.

## Mitigating Hot Keys: Key Salting

Most databases cannot automatically split traffic for a single hot key. An application-level workaround is **key salting**:

- Append a random suffix (e.g., two digits `00`–`99`) to the hot key on write, spreading writes across 100 distinct keys and partitions.
- On read, query all 100 salted keys and merge the results.

Because salting adds significant read fan-out and bookkeeping, apply it **only to a small, known set of hot keys** (for example, tracked via a hot-key registry or feature flag), not to every key in the database.

> [!warning] Failure modes & trade-offs
> - **Read amplification**: Salting a key across $K$ buckets cuts write load per partition by $K$, but multiplies read requests by $K$ and exposes reads to scatter/gather tail latency.
> - **Uniqueness & atomic counters break**: Salted keys live on different partitions, so you lose single-partition atomic uniqueness checks and must aggregate counts asynchronously.

What is the difference between **skew** and a **hot spot** in data partitioning?::**Skew** is an uneven distribution of data or query load across partitions; a **hot spot** is a specific partition receiving disproportionately high load that bottlenecks the cluster.
^q-jyyu

## Why can't hash partitioning eliminate celebrity hot keys, and how does key salting mitigate them? #card
A deterministic hash function always maps identical keys to the exact same partition, so millions of writes to one celebrity ID still hit one node. **Key salting** appends a small random suffix (e.g., `00`–`99`) to spread writes across multiple partitions, at the cost of reading and merging all salted variants on read.
^q-yqdc

Apply key salting only to a tracked subset of known hot keys rather than globally, to avoid paying unnecessary read fan-out across normal traffic. #card-reminder
^q-qz37

## Related
- [[Data partitioning]]
- [[Key-Range Partitioning]]
- [[Hash Partitioning]]
- [[Partitioning and Secondary Indexes]]
- [[Rebalancing Partitions]]
- [[Horizontal scaling]]
