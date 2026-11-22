---
flashcards:
  q-rvnv: { nid: 1795356726018, hash: f6st6can, sync: twg8i8gt }
  q-pje4: { nid: 1795356726044, hash: j6zy9nvf, sync: 65239yn5 }
---

In the early days of business data processing, a write to the database typically corresponded to a *commercial transaction* taking place: making a sale, placing an order with a supplier, paying an employee’s salary, etc. As databases expanded into areas that didn’t involve money changing hands, the term transaction nevertheless stuck, referring to a group of reads and writes that form a logical unit.

A transaction needn’t necessarily have [[ACID]] properties. Transaction processing just means allowing clients to make low-latency reads and writes ([[OLTP]]) — as opposed to batch processing jobs, which only run periodically (for example, once per day).

## Single-Node vs. Distributed Transactions

- **Single-node transactions:** Enforced locally by one [[DBMS]] using a [[WAL]], [[MVCC]], and [[Database Lock|locks]]. Fast, well-understood, and cheap when kept short.
- **Distributed transactions:** Spanning multiple shards or microservices requires coordination protocols like [[Two-Phase Commit]] (blocking, latency-sensitive) or asynchronous compensating workflows like [[Saga]] (see [[2PC vs Saga]]).

> [!warning] Failure modes & trade-offs
> - **Long-running transactions hurt the entire database:** Keeping a transaction open while calling an external HTTP API holds [[Database Lock|locks]], pins database connections, and stalls [[MVCC]] garbage collection (`VACUUM`). Always perform external I/O *outside* the transaction boundary (or use the [[Outbox Pattern]]).
> - **Retryability is not automatic:** Serialization failures (`40001`), deadlocks (`40P01`), and network drops after commit require application-level retries combined with [[Idempotency Key|idempotency keys]] to avoid duplicate side effects.

## Flashcards

Does a database [[Transaction]] always imply full [[ACID]] guarantees?::No — a transaction is simply a logical group of reads and writes; the exact safety guarantees depend on the engine and configured isolation level.
^q-rvnv

Never perform slow external HTTP calls inside an open database transaction; it pins connections, holds locks, and blocks [[MVCC]] cleanup. #card-reminder
^q-pje4

## Related

- [[ACID]]
- [[MVCC]]
- [[OLTP]]
- [[Database Lock]]
- [[Snapshot Isolation]]
- [[Write Skew]]
- [[Two-Phase Commit]]
- [[2PC vs Saga]]
- [[Outbox Pattern]]