**SSTables (Sorted String Tables)** are the foundational disk-based building blocks of an [[LSM Tree]].

**Definition:** An SSTable is an immutable, sorted file of key-value pairs. Once an SSTable is written to disk, it **never changes**.

- An [[LSM Tree]] is essentially a hierarchy and management strategy for multiple SSTables across different levels on disk, combined with an in-memory buffer ([[LSM Tree#Memtable]]).
- Every time a Memtable is flushed to disk, a **new SSTable is born**.
- Because data is continuously written, an LSM-Tree accumulates many individual SSTables over time, which are periodically merged and deduplicated via background compaction.

## Internal Layout of an SSTable

A typical SSTable (e.g., RocksDB / [[Apache Cassandra]]) is organized into:
1. **Compressed data blocks** — contiguous sorted key-value pairs (typically 4–64 KB blocks).
2. **Sparse index** — stores the offset of the first key of each block (fits in memory; binary search locates the block, then a short scan finds the exact key).
3. **[[Bloom Filter]]** — per-SSTable filter to skip reading the file entirely when a queried key is absent.
4. **Metadata / footer** — checksums, compression codec, and pointers to index/filter blocks.

> [!warning] Failure modes & trade-offs
> - **Tombstone accumulation**: Because SSTables are immutable, deletes are written as tombstones. Heavy delete workloads (e.g., queue-like tables in [[Apache Cassandra]]) can pile up thousands of tombstones between compactions, causing range scans to read huge numbers of dead entries and time out.
> - **Unbounded SSTable growth without compaction**: If compaction is throttled or disabled, read latency degrades linearly with the number of SSTable files that must be checked.

SSTable (Sorted String Table):::An immutable on-disk file of key-value pairs sorted by key, paired with a sparse index and typically a [[Bloom Filter]].

## Why can an SSTable use a sparse in-memory index instead of indexing every key? #card
Because keys are stored in sorted order on disk. Knowing the byte offsets of a few boundary keys (e.g., one per 4–64 KB compressed block) lets the engine jump to the bounding block and quickly scan within it to find any key in between.

Because SSTables are immutable, updates and deletes never overwrite existing files; instead they ==append a newer key-value entry or a tombstone to the memtable== and reclaim old versions during compaction.

## Related
- [[LSM Tree]]
- [[Bloom Filter]]
- [[Database Log]]
- [[WAL]]
- [[Apache Cassandra]]
- [[DDIA Chapter 3 Storage and Retrieval]]
