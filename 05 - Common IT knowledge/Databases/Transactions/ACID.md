A set of four guarantees that traditional [[RDBMS|relational databases]] (and some modern [[NoSQL database|NoSQL]] stores) provide for [[Transaction|transactions]] so applications can reason safely about concurrent access and crashes.

## The Four Properties

- **Atomicity (abortability):** All writes in a transaction either succeed together or are rolled back together — never left half-applied if the process crashes or an error occurs mid-flight. *(Note: in ACID, atomicity is about fault recovery, not concurrency.)*
- **Consistency:** Application-defined invariants (e.g., debits equal credits, foreign keys remain valid) are preserved from one valid state to the next. Unlike A, I, and D, consistency is primarily an **application responsibility**; the database can only enforce declared constraints (foreign keys, `UNIQUE`, `CHECK`).
- **Isolation:** Concurrent transactions do not step on each other's toes. True serializability means the outcome is identical to running transactions one after another, though most databases default to weaker isolation levels for performance.
- **Durability:** Once a transaction commits, its data survives hardware crashes and power loss — typically by flushing changes to a [[WAL|write-ahead log]] (`fsync`) and/or replicating them to other nodes before acknowledging the client.

## Isolation Levels & Anomalies

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | [[Write Skew]] |
| :--- | :--- | :--- | :--- | :--- |
| **Read Uncommitted** | Possible | Possible | Possible | Possible |
| **Read Committed** *(PostgreSQL default)* | Prevented | Possible | Possible | Possible |
| **Repeatable Read / [[Snapshot Isolation]]** *(MySQL default)* | Prevented | Prevented | Prevented in PG MVCC | Possible |
| **Serializable** | Prevented | Prevented | Prevented | Prevented |

> [!warning] Failure modes & trade-offs
> - **"ACID" is marketing unless you check the isolation level:** PostgreSQL's default `Read Committed` still allows lost updates and [[Write Skew]]. Even `Repeatable Read` ([[Snapshot Isolation]]) allows [[Write Skew]] unless you use `SELECT ... FOR UPDATE` ([[Pessimistic Locking]]) or `SERIALIZABLE` (SSI).
> - **Durability vs. latency:** Disabling `synchronous_commit` or relying on async [[Replication]] improves write latency, but acknowledged commits can be lost on a crash or failover.
> - **Cross-service ACID doesn't scale:** Distributed [[Two-Phase Commit]] hurts availability and latency across microservices; prefer [[Saga]] + [[Outbox Pattern]] + [[Idempotency]] instead (see [[2PC vs Saga]]).

## Flashcards

In ACID, what does **Atomicity** actually guarantee?::Abortability — if a transaction fails or crashes partway through, all of its partial writes are rolled back so the client can safely retry.

Why is the **C (Consistency)** in ACID different from A, I, and D?::Consistency depends on application-level invariants (the database only enforces constraints like `UNIQUE` or `CHECK`), whereas A, I, and D are properties of the database engine itself.

Even under [[Snapshot Isolation]] (PostgreSQL `REPEATABLE READ`), concurrent transactions can still suffer from ==[[Write Skew]]== unless prevented by explicit locking or `SERIALIZABLE` isolation.

## How do databases guarantee Durability without flushing random data pages on every commit? #card

- They append changes sequentially to a [[WAL|Write-Ahead Log]] and call `fsync` before acknowledging the commit.
- Dirty heap/index pages are flushed to disk lazily in the background; if a crash occurs, the engine replays the [[WAL]] on startup to recover committed transactions.

## Related

- [[Transaction]]
- [[MVCC]]
- [[Snapshot Isolation]]
- [[Write Skew]]
- [[WAL]]
- [[Database Lock]]
- [[Two-Phase Commit]]
- [[2PC vs Saga]]
- [[RDBMS]]
