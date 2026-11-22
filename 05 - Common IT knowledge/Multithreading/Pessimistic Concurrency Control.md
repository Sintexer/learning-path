---
flashcards:
  q-ig8z: { nid: 1795356718166, hash: up7n9sji, sync: 66gaviry }
  q-4pgf: { nid: 1795356718192, hash: ew6vnpc3, sync: 2g4j5tn9 }
---

Pessimistic strategies involve locking the data before you even touch it, ensuring that no other process can modify or even read it until you are finished. If another process attempts to access the resource while it is locked, it is forced to wait or is blocked entirely. This approach assumes that conflicts are likely to happen, so it prioritizes safety and consistency over speed. While this prevents the need for retries, it introduces the risk of deadlocks and reduces overall system throughput because processes spend significant time waiting for locks to be released.

If you work with a [[Database|database]] system that uses `SELECT FOR UPDATE` ([[Pessimistic Locking]]) or a file system that prevents multiple users from opening a file for writing, you are using this algorithm. Pessimistic strategies are essential for physical resources - like a printer or a network socket - because you cannot "copy" the hardware to modify it safely; you must control access to the single, real-world instance.

> [!warning] Failure modes & trade-offs
> - **Deadlocks:** Two transactions locking rows `A` then `B` vs. `B` then `A` will deadlock; enforce a deterministic lock ordering and keep transactions short.
> - **Throughput collapse under long locks:** Holding pessimistic locks across external HTTP/RPC calls stalls every contending transaction and exhausts the connection pool.
> - **When to switch to optimistic:** If read-heavy workloads have rare write conflicts, [[Optimistic Concurrency Control]] (using a [[Version Guard]] or [[Compare-And-Swap|CAS]]) avoids blocking and scales much better.

## Flashcards

When should you prefer [[Pessimistic Concurrency Control]] over [[Optimistic Concurrency Control]]?::When write contention is high (retries would thrash), rollback/retry is expensive or impossible (e.g. physical/external side effects), or strict serialized access is required.
^q-ig8z

Pessimistic concurrency control assumes conflicts are ==likely== and acquires locks before access; [[Optimistic Concurrency Control]] assumes conflicts are ==rare== and validates at commit/swap time.
^q-4pgf

## Related
- [[Pessimistic Locking]]
- [[Optimistic Concurrency Control]]
- [[Mutual Exclusion]]
- [[Critical Section]]
- [[Database Lock]]
- [[MVCC]]