Primary-key lookups in a [[Data partitioning|partitioned]] database are straightforward: hash or range-lookup the key and route directly to the owning partition. **Secondary indexes** (e.g., querying all cars where `color = 'red'` or `make = 'Honda'`) do not map cleanly to a single partition because matching records live across many partitions.

Distributed databases handle secondary indexes in two fundamental ways:

## 1. Document-Partitioned (Local) Secondary Indexes

Each partition maintains its **own independent secondary index** covering *only* the documents stored in that partition.

- **Writes**: Fast and local — inserting or updating a document only touches the single partition holding that document (and its local index).
- **Reads**: Expensive **scatter/gather** — to query `WHERE color = 'red'`, the coordinator must broadcast the query to *all* partitions in parallel and merge the results.
- **Examples**: [[MongoDB]], [[Apache Cassandra]] (local secondary indexes), Elasticsearch, Riak, VoltDB.

## 2. Term-Partitioned (Global) Secondary Indexes

A single **global index** covers data across all partitions, and the index itself is partitioned by the indexed term (e.g., `color` A–M on Partition 0, N–Z on Partition 1).

- **Reads**: Fast and targeted — querying `WHERE color = 'red'` routes to only the partition(s) owning the term `'red'`, avoiding scatter/gather.
- **Writes**: Complex and cross-partition — updating one document may affect several indexed fields (`color`, `make`, `year`), requiring updates to multiple remote index partitions.
- **Sync vs Async updates**: Updating all affected global index partitions atomically requires a distributed transaction ([[Two-Phase Commit]]). Therefore, most production systems (e.g., Amazon DynamoDB GSIs, Riak search) update global indexes **asynchronously**, meaning index reads are [[Eventual Consistency|eventually consistent]] and may briefly lag behind the primary table.

## Comparison

| Dimension | Local (Document-Partitioned) | Global (Term-Partitioned) |
|---|---|---|
| **Write path** | Single partition (fast, atomic) | Multi-partition (slow or async) |
| **Read path** | Scatter/gather across all partitions | Single/few partitions (fast) |
| **Consistency** | Immediately consistent with local partition | Usually eventually consistent (async) |
| **Tail latency risk** | High (amplified across all partitions) | Low on reads; hotspot risk on popular terms |

> [!warning] Failure modes & trade-offs
> - **Scatter/gather tail-latency amplification**: With local indexes, querying 100 partitions means every read waits for the *slowest* of the 100 partitions (see [[Reliability, Scalability, Maintainability]]). Even with 1% slow partitions, most queries hit tail latency.
> - **Hot index partitions**: With global indexes, a low-cardinality term (e.g., `status = 'ACTIVE'`) routes all matching index writes and reads to a single index partition, creating a severe hotspot.

## Compare local (document-partitioned) vs global (term-partitioned) secondary indexes. #card
- **Local index**: Each partition indexes only its own documents. **Fast local writes**, but **reads require scatter/gather** across all partitions (high tail latency).
- **Global index**: Index is partitioned by the indexed term across nodes. **Fast targeted reads**, but **writes span multiple partitions** and are typically updated asynchronously ([[Eventual Consistency|eventually consistent]]).

Why are global (term-partitioned) secondary indexes usually updated asynchronously in distributed databases?::Updating the primary record and multiple remote index partitions synchronously would require a slow, fragile distributed transaction ([[Two-Phase Commit]]).

In a document-partitioned secondary index, every query must perform a ==scatter/gather== across all partitions, exposing reads to tail-latency amplification.

## Related
- [[Data partitioning]]
- [[Database Index]]
- [[Consistent Hashing]]
- [[Two-Phase Commit]]
- [[Eventual Consistency]]
- [[Apache Cassandra]]
- [[MongoDB]]
