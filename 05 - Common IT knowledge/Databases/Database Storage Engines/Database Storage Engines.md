---
flashcards:
  q-77kc: { nid: 1795356726667, hash: nka3tfbw, sync: tve3x3tj }
---

Database engines mainly diverged in two directions:
- [[OLTP]] oriented
- [[OLAP]] oriented

Let's clarify the terminology:

[[RDBMS]] / [[DBMS]] layer:
- Handles network protocol / client connections
- [[SQL]] parser & query optimizer
- [[Authentication]] & permissions

Storage Engine layer:
- Writes/reads bytes to and from disk/SSD (e.g., [[B-Tree Index|B-Trees]], [[LSM Tree]]s, or [[Column-Oriented Storage]])
- Manages the Buffer Pool (RAM cache)
- Manages the Transaction Log ([[WAL]] / Redo Log)
- Handles row [[Database Lock|locking]], internal [[Database Latch|latches]], and [[MVCC]] concurrency

**MySQL is completely decoupled:** MySQL is the [[DBMS]] shell. You can choose different storage engines per table:
- `InnoDB`: Supports [[ACID]] transactions, row-level locking, foreign keys (the default, uses clustered [[B-Tree Index|B-Trees]]).
- `MyISAM`: Fast reads, no transactions, table-level locking.
- `Memory`: Stores everything strictly in RAM (see [[In-Memory Database]]).
- `RocksDB (MyRocks)`: Optimized for high-write SSD workloads using [[LSM Tree]]s.

**Is [[PostgreSQL]] decoupled?**
- Historically: **No.** PostgreSQL was tightly coupled to its own heap storage engine.
- Modern PostgreSQL (v12+): Introduced **Table Access Methods (TAM)**. The engine is now decoupled enough that extensions can plug in completely different storage engines. For example:
	- `Citus`: Turns Postgres into a distributed database ([[Data partitioning|sharding]]).
	- `pg_analytics` / `Hydra`: Plugs a [[Column-Oriented Storage|columnar storage engine]] directly into PostgreSQL.

## Simplest database case

The world's simplest database could be implemented as two bash functions: one writes a key-value pair to a file, and the other reads the last match for a key. Such a database has incredibly good write performance ($O(1)$ append). It is effectively the easiest and most naive implementation of a [[Database Log]]. However, it has terrible read performance as it has to scan $O(N)$ to find the matching record.

That is why real databases need a [[Database Index|index]] — a data structure for finding the value for a particular key efficiently, at the cost of extra write overhead to maintain the index.

> [!tip] Production reality
> Choosing a storage engine is choosing your amplification profile:
> - **Page-oriented ([[B-Tree Index|B-Tree]])**: Predictable read latency and strong transactional locking (each key lives in one place), but higher random-write I/O and page fragmentation.
> - **Log-structured ([[LSM Tree]])**: High sequential write throughput and better disk compression, but compaction causes background I/O contention and tail-latency spikes.
> - **[[Column-Oriented Storage|Columnar]]**: Massive scan speedups and compression for [[OLAP]], but poor for single-row updates.

## What responsibilities belong to the Storage Engine vs the RDBMS shell? #card
- **[[RDBMS]] shell**: Client connections, network protocol, [[SQL]] parsing, query optimization, [[Authentication]]/[[Authorization]].
- **Storage Engine**: On-disk layout (B-Tree/LSM/columnar), buffer pool management, [[WAL]]/crash recovery, [[Database Latch|latches]], row [[Database Lock|locks]], and [[MVCC]].

Every [[Database Index]] speeds up reads at the cost of ==slower writes==, because every write must update both the data and its index structures.

What [[PostgreSQL]] (v12+) feature allows plugging in custom storage engines?::Table Access Methods (TAM)
^q-77kc

## Related
- [[Database Log]]
- [[WAL]]
- [[Database Latch]]
- [[Database Index]]
- [[LSM Tree]]
- [[B-Tree Index]]
- [[Column-Oriented Storage]]
- [[OLTP and OLAP]]
- [[DDIA Chapter 3 Storage and Retrieval]]
