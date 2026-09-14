
Database engines mainly diverged in two directions:
- [[OLTP]] oriented
- [[OLAP]] oriented

Let's clarify the terminology:

RDBMS:
- Handles Network protocol / Client connections  
- SQL Parser & Query Optimizer 
- Authentication & Permissions

Storage Engine:
- Writes/reads bytes to and from the disk/SSD  
- Manages the Buffer Pool (RAM cache)
- Manages the Transaction Log ([[WAL]] / Redo Log)
- Handles row locking and [[MVCC]] concurrency

**MySQL is completely decoupled:** MySQL is the DBMS shell. You can choose different storage engines per table:
- `InnoDB`: Supports ACID transactions, row-level locking, foreign keys (the default).
- `MyISAM`: Fast reads, no transactions, table-level locking.
- `Memory`: Stores everything strictly in RAM.
- `RocksDB (MyRocks)`: Optimized for high-write SSD workloads using LSM-trees.

**Is PostgreSQL decoupled?**
- Historically: **No.** PostgreSQL was tightly coupled to its own heap storage engine.
- Modern PostgreSQL (v12+): Introduced **Table Access Methods (TAM)**. The engine is now decoupled enough that extensions can plug in completely different storage engines. For example:
	- `Citus`: Turns Postgres into a distributed database.
	- `pg_analytics` / `Hydra`: Plugs a columnar storage engine directly into PostgreSQL.

## Simplest database case 

The world's simplest database could be implemented as two bash functions: one write a key-value pair to a file, and the other reads the first match for a key. Such database has an incredibly good performance for write. It is effectively an easiest and most naive implementation of the [[Database Log]]. However it also has a bad read performance as it has to iterate $O(N)$ to find the matching record.

That is why real databases need an [[Database Index|index]] - a data structure for finding the value for a particular key efficiently. 