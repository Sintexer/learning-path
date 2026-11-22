Multi-Version Concurrency Control

In older database systems, if Transaction A was **reading** a table, it locked the table, and Transaction B had to **wait** before it could update a row. If Transaction B was **writing**, readers were blocked. Concurrency was terrible.

**MVCC solves this with a golden rule:**

> _"Readers never block writers; writers never block readers."_

## How MVCC physically works:

When you execute an `UPDATE` in [[PostgreSQL]], the engine **does not overwrite the old data on disk.**

Instead:

1. It writes a **brand new row version (tuple)** to an open disk page.
2. It leaves the old row version sitting right where it was, but marks it as obsolete.

Every row has hidden system headers:

- `xmin`: The transaction ID that **created** this row version.
- `xmax`: The transaction ID that **deleted or replaced** this row version.

```
Initial State (Row inserted by Transaction 100):
[ TID (1,1) | xmin: 100 | xmax: 0    | name: "Alice" ]

Transaction 105 runs: UPDATE users SET name = 'Bob' WHERE name = 'Alice';
Disk layout now:
[ TID (1,1) | xmin: 100 | xmax: 105  | name: "Alice" ] <-- marked as killed by 105
[ TID (1,2) | xmin: 105 | xmax: 0    | name: "Bob"   ] <-- new active version
```

If an older, long-running Transaction 102 queries `Alice`, it looks at `xmax: 105`, realizes 105 committed _after_ it started, and safely reads the old data!

**The downside:** These dead rows accumulate on disk. This is called **bloat**.

**The solution:** A background process called [[PostgreSQL#VACUUM]] cleans up obsolete rows whose `xmax` is older than all currently running transactions.

> [!warning] Failure modes & trade-offs
> - **PostgreSQL vs. MySQL InnoDB MVCC:** PostgreSQL keeps old row versions *inline in the main heap* (requiring `VACUUM` and risking table/index bloat). InnoDB overwrites rows in place and stores old versions in a separate **undo log** — avoiding heap bloat, but making rollbacks and very long reads traverse long undo chains.
> - **[[Snapshot Isolation]] still allows [[Write Skew]]:** MVCC ensures readers see a consistent snapshot without blocking writers, but two concurrent transactions can read the same snapshot and update *disjoint* rows in a way that violates a multi-row invariant (e.g., two doctors going off-call simultaneously).
> - **Long-running transactions stall cleanup:** A single forgotten `IDLE IN TRANSACTION` session prevents `VACUUM` (or InnoDB purge) from freeing dead versions across the entire database.

## Flashcards

What is the golden rule of MVCC?::Readers never block writers, and writers never block readers.

In [[PostgreSQL]] MVCC, ==`xmin`== records the transaction ID that created a tuple, while ==`xmax`== records the transaction ID that deleted or replaced it.

How does [[PostgreSQL]]'s MVCC storage differ from MySQL InnoDB's?::PostgreSQL stores old tuple versions inline in the heap (cleaned by `VACUUM`), whereas InnoDB overwrites in place and stores older versions in a separate undo log.

## Related

- [[PostgreSQL]]
- [[ACID]]
- [[Transaction]]
- [[Snapshot Isolation]]
- [[Write Skew]]
- [[Database Lock]]