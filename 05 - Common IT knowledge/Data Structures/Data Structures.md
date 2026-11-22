Core in-memory and on-disk data structures used across [[Database Storage Engines]], [[Database Index|database indexes]], and distributed systems.

## On-Disk & Storage Engine Structures
- [[LSM Tree]] — write-optimized log-structured merge tree (memtable + immutable [[SSTable]]s + background compaction).
- [[SSTable]] — sorted, immutable key-value segment file; building block of [[LSM Tree]]s.
- [[B-Tree Index]] — update-in-place balanced page tree; default index structure in most [[RDBMS]] engines.
- [[Hash Index]] — in-memory hash table mapping keys to byte offsets in an append-only [[Database Log]].
- [[Database Log]] / [[WAL]] — append-only sequential files for durability and crash recovery.

## Probabilistic & Bit-Level Structures
- [[Bitmap]] — compact bit arrays used in [[Bitmap Index|bitmap indexes]], visibility maps, and heap scans.
- [[Bloom Filter]] — probabilistic set-membership structure with zero false negatives; avoids wasted disk reads in [[LSM Tree]]s.

## In-Memory Balanced Trees (Memtable Internals)
- [[AVL Tree]] — strictly balanced BST (faster lookups, more rotations on write).
- [[Red-Black Tree]] — loosely balanced BST (fewer rotations on insert/delete; common for memtables alongside skip lists).

> [!tip] Production reality
> Choosing a storage data structure is fundamentally a trade-off along the **RUM Conjecture**: you can optimize at most two of **Read overhead**, **Update (write) overhead**, and **Memory/space overhead** — never all three simultaneously. [[B-Tree Index|B-Trees]] favor read latency; [[LSM Tree]]s favor write throughput and space compaction.

What are the three axes of the RUM Conjecture in storage data structures?::Read overhead, Update (write) overhead, and Memory/space overhead (optimizing any two degrades the third).

## Related
- [[Database Storage Engines]]
- [[Database Index]]
- [[DDIA Chapter 3 Storage and Retrieval]]
- [[Algorithms TODO]]
