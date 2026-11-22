> AKA **Relational Database Management System**

With a **relational database management system (RDBMS)**, you can create, update, and administer a [[Relational Database|relational database]]. It is a [[DBMS]] built around the relational model (tables, rows, columns, and declarative constraints).

You communicate with an RDBMS using Structured Query Language ([[SQL]]).

## Core Capabilities

- **Schema-on-write & integrity constraints:** Enforces types, `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, and `CHECK` constraints at the database boundary so invalid data cannot silently corrupt state.
- **[[ACID]] [[Transaction|transactions]]:** Combines [[WAL|write-ahead logging]], [[MVCC]], and [[Database Lock|locking]] to guarantee crash recovery and configurable isolation levels.
- **Cost-based query optimizer:** Translates declarative [[SQL]] into physical execution plans (choosing [[B-Tree Index|indexes]], hash/merge/nested-loop joins, and [[Grouping|aggregation strategies]]) based on table statistics.
- **Popular engines:** [[PostgreSQL]], MySQL (InnoDB), Oracle Database, Microsoft SQL Server, SQLite.

> [!warning] Failure modes & trade-offs
> - **Write-scaling ceiling on a single primary:** Traditional RDBMSs scale reads easily via [[Leader-Based Replication|read replicas]], but all writes flow through one primary node. Beyond vertical scaling ([[Vertical scaling]]), write scaling requires [[Data partitioning|sharding]] or moving to a distributed SQL engine (CockroachDB, Spanner).
> - **Schema evolution requires discipline:** Because schemas are enforced on write, careless `ALTER TABLE` statements lock tables or break rolling deployments unless you follow zero-downtime [[Schema Migration]] patterns.
> - **Connection overhead:** Each client connection carries memory/process overhead — production services need connection pooling (e.g., HikariCP, PgBouncer).

## Flashcards

How does a traditional [[RDBMS]] scale reads versus writes?::Reads scale horizontally via [[Leader-Based Replication|read replicas]] (accepting [[Replication Lag]]); writes initially scale via [[Vertical scaling]] on the single primary, and beyond that require [[Data partitioning|sharding]] or distributed SQL.

What is the main advantage of an RDBMS's **schema-on-write** model compared to schema-on-read?::The database enforces types and constraints (`UNIQUE`, `FOREIGN KEY`, `CHECK`, `NOT NULL`) at write time, preventing corrupt or inconsistent shapes from ever entering storage.

## Related

- [[DBMS]]
- [[Relational Database]]
- [[PostgreSQL]]
- [[SQL]]
- [[ACID]]
- [[MVCC]]
- [[Schema Migration]]
- [[SQL vs NoSQL]]
