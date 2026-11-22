---
flashcards:
  q-zk2f: { nid: 1795356729418, hash: 6ahx2jyn, sync: eg3pcthm }
  q-8f8e: { nid: 1795356729451, hash: djwjkwf2, sync: ka2atczq }
---

Databases handle data storage and retrieval using two main approaches optimized for different workloads: **[[OLTP]]** (transaction processing) and **[[OLAP]]** (analytics). See also [[OLTP and OLAP]] and [[Database Storage Engines]].

### [[OLTP]] (Online Transaction Processing)

**Use case:** User-facing applications (banking, e-commerce, social media)

- High volume of requests
- Each query touches a small number of records
- Access pattern: key-based lookups
- **Bottleneck:** Disk seek time
- **Optimization:** [[Database Index|Indexes]] to find specific data quickly

### [[OLAP]] (Online Analytical Processing)

**Use case:** [[Data Warehouse|Data warehouses]], business analytics

- Low volume of queries (but each is demanding)
- Scans millions of records per query
- Access pattern: sequential scans across large datasets
- **Bottleneck:** Disk bandwidth
- **Optimization:** [[Column-Oriented Storage|Column-oriented storage]] to minimize data read from disk

### OLTP Storage Engines: Two Schools

#### 1. Log-Structured

Never updates existing files; only appends and deletes obsolete files via compaction.

**Key idea:** Convert random-access writes into sequential writes for higher throughput.

**Examples:** Bitcask ([[Database Log]] + [[Hash Index]]), [[SSTable]]s, [[LSM Tree|LSM-trees]], LevelDB, RocksDB, [[Apache Cassandra]], HBase, Lucene

#### 2. Update-in-Place (Page-Oriented)

Treats disk as fixed-size pages (typically 4–16 KB) that can be overwritten, guarded by a [[WAL]] for crash recovery and [[Database Latch|latches]] for concurrent access.

**Examples:** [[B-Tree Index|B-trees]] (used in most [[Relational Database|relational]] and many nonrelational databases)

### OLAP Optimization: [[Column-Oriented Storage]]

Traditional row-oriented databases store entire rows together. [[Column-Oriented Storage|Column-oriented storage]] stores each column separately, enabling:

- Compact encoding of data (run-length encoding, [[Bitmap]] encoding)
- Reading only the columns needed for a query
- Vectorized execution and significant speed improvements for analytical workloads

### Key Takeaways

| Aspect | [[OLTP]] | [[OLAP]] |
|---|---|---|
| **Workload** | Many small, fast queries | Few large, complex queries |
| **Data access** | Key-based lookups | Sequential scans |
| **Critical resource** | Disk seek time | Disk bandwidth |
| **Storage focus** | Indexes, fast writes | Compression, read efficiency |
| **Best storage engine** | [[B-Tree Index\|B-trees]] or [[LSM Tree\|log-structured]] | [[Column-Oriented Storage\|Column-oriented]] |

> [!tip] Production reality
> B-Trees vs LSM-Trees is not "old vs new":
> - **[[B-Tree Index|B-Trees]]** offer more predictable read latencies and simpler transactional isolation (each key exists in exactly one place on disk).
> - **[[LSM Tree|LSM-Trees]]** sustain much higher write throughput and achieve better disk compression, but background compaction can cause p99 latency spikes.

## What is the primary hardware bottleneck for OLTP vs OLAP workloads? #card
- **[[OLTP]]**: Disk seek time / random IOPS (many small key lookups) — optimized with [[Database Index|indexes]].
- **[[OLAP]]**: Disk bandwidth (scanning millions of rows) — optimized with [[Column-Oriented Storage|columnar storage]] and compression.

The two main schools of [[OLTP]] storage engines are ==log-structured (LSM-Trees / SSTables)== and ==page-oriented update-in-place (B-Trees)==.
^q-zk2f

## Why does column-oriented storage compress much better than row-oriented storage? #card
Values within a single column share the same data type and often have low cardinality or repeating runs (especially when sorted), making run-length encoding and [[Bitmap]] compression dramatically more effective than across heterogeneous row fields.
^q-8f8e

## Related
- [[Database Storage Engines]]
- [[Database Log]]
- [[WAL]]
- [[Database Index]]
- [[Database Index Practical Usage]]
- [[LSM Tree]]
- [[B-Tree Index]]
- [[Column-Oriented Storage]]
- [[Transaction]]
- [[OLTP and OLAP]]
- [[Star Schema vs Snowflake Schema]]
- [[In-Memory Database]]
- [[Data Warehouse]]
- [[ETL]]
