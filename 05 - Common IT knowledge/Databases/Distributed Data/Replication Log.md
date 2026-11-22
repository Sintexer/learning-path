This is how data changes are streamed from [[Leader-Based Replication#Leader|leader]] to [[Leader-Based Replication#Follower|followers]].

## Statement-based replication

The simplest case: the leader logs every write request (`INSERT`, `UPDATE`, `DELETE` statement) that it executes and sends that statement log to its followers. This indirectly breaks on nondeterministic statements like `RAND()`, `NOW()`, auto-incrementing columns, triggers, and stored procedures, which can produce different results on different replicas and silently cause divergence.

## Write-ahead log (WAL) shipping

Default for physical replication in [[PostgreSQL]] (and Oracle). The DB ships low-level [[WAL]] entries containing exact disk block/byte modifications. This tightly couples replication to the [[Database Storage Engines|storage engine]] internals: if the database changes its on-disk storage format between major versions, you typically cannot run different database versions on the leader and followers (blocking [[Zero-downtime deployment|zero-downtime]] major upgrades).

## Logical (row-based) log replication

An alternative is to use a separate log format for replication, decoupled from the storage engine's physical layout. A **logical log** describes changes at the row granularity:
- For an `INSERT`: the new values of all columns.
- For a `DELETE`: enough information (primary key) to uniquely identify the deleted row.
- For an `UPDATE`: row identifier plus the new values of changed columns.

Because logical logs are decoupled from storage internals, leader and followers can run **different DB versions** (enabling zero-downtime upgrades) and external systems can parse the stream directly — this is the foundation of **[[Change Data Capture]] (CDC)** (e.g., Debezium reading MySQL binlog or PostgreSQL logical replication slots).

## Trigger-based replication

When more flexibility is needed — replicating a subset of tables, replicating across heterogeneous databases, or running custom conflict resolution — replication can move up to the application layer via database triggers or external log-reading tools (e.g., Oracle GoldenGate, Bucardo for [[PostgreSQL]]).

A trigger registers custom code that fires on every write transaction and logs the change into a separate table, which an external process reads and forwards (conceptually similar to the [[Outbox Pattern]]). Trigger-based replication carries higher write overhead and more operational fragility than built-in logical replication, so prefer logical replication / [[Change Data Capture]] when available.

> [!warning] Failure modes & trade-offs
> - **WAL shipping blocks rolling DB upgrades**: Physical WAL replication requires identical major versions and OS/CPU architectures on leader and follower. Use logical replication for major-version upgrades.
> - **Logical replication slot bloat in [[PostgreSQL]]**: If a downstream [[Change Data Capture|CDC]] consumer stops reading from a logical replication slot, Postgres retains WAL segments indefinitely until the primary disk fills up and crashes the database. Always monitor replication slot lag and set `max_slot_wal_keep_size`.

## Compare WAL shipping vs Logical (row-based) replication. #card
- **[[WAL]] shipping**: Streams physical disk-block byte changes. Fast, minimal overhead, but tightly coupled to storage engine version — prevents mixed-version rolling upgrades and external consumption.
- **Logical (row-based) replication**: Streams row-level insert/update/delete events decoupled from disk layout. Enables [[Zero-downtime deployment|zero-downtime major upgrades]] and powers [[Change Data Capture]] (CDC).

Why is statement-based replication fragile in production?::Nondeterministic functions (`NOW()`, `RAND()`), auto-increment ordering, and triggers can evaluate differently on leader vs follower, silently causing replicas to diverge.

In [[PostgreSQL]], an abandoned logical replication slot can crash the primary database because ==Postgres retains unconsumed WAL segments indefinitely until disk fills up==.

## Related
- [[Leader-Based Replication]]
- [[WAL]]
- [[Database Log]]
- [[Change Data Capture]]
- [[Outbox Pattern]]
- [[PostgreSQL]]
- [[Zero-downtime deployment]]
