---
flashcards:
  q-8hyy: { nid: 1795356727567, hash: xjfc69jn, sync: k9rbwa6s }
  q-j33r: { nid: 1795356727591, hash: jbfi8hd8, sync: 839pvdv7 }
  q-6j9u: { nid: 1795356727617, hash: u6n6dwzf, sync: k24wgy9d }
---

Row-level locks are used for **Pessimistic Locking**: when you anticipate race conditions and want to prevent two concurrent processes from modifying the same data at the same time.

[[Optimistic Locking]] assumes conflicts are **rare**, so it doesn't lock anything upfront. Instead, it lets transactions proceed freely and only checks for conflicts **at commit/update time**.

## How Pessimistic Locking Works in DBs

**DB-level mechanics:**

- When a transaction reads a row with intent to update, it acquires a **row-level lock** (shared or exclusive) via `SELECT ... FOR UPDATE` (or `FOR SHARE` for read locks).
- Other transactions trying to acquire a conflicting lock on the same row must **wait** until the lock holder commits or rolls back.
- Locks are tracked internally in a **lock manager** — a table mapping resource IDs (rows, pages, tables) to lock holders and modes (shared/exclusive).
- Deadlocks are detected (via wait-for graphs) and one transaction is aborted to break the cycle.
- Variants: `FOR UPDATE NOWAIT` (fail immediately if locked) and `FOR UPDATE SKIP LOCKED` (skip locked rows — useful for queue-like processing).

## Controlling It from Java (JPA/Hibernate)

**1. Using `EntityManager` with lock mode:**

```java
Account acc = em.find(Account.class, id, LockModeType.PESSIMISTIC_WRITE);
```

This issues `SELECT ... FOR UPDATE` under the hood.

**2. Explicit lock on an already-loaded entity:**

```java
em.lock(acc, LockModeType.PESSIMISTIC_WRITE);
```

**3. Using JPQL query hints:**

```java
TypedQuery<Account> q = em.createQuery(
    "SELECT a FROM Account a WHERE a.id = :id", Account.class);
q.setParameter("id", id);
q.setLockMode(LockModeType.PESSIMISTIC_WRITE);
Account acc = q.getSingleResult();
```

**4. Spring Data JPA:**

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT a FROM Account a WHERE a.id = :id")
Account findByIdForUpdate(@Param("id") Long id);
```

**Common `LockModeType` values:**

- `PESSIMISTIC_READ` → shared lock (`FOR SHARE`)
- `PESSIMISTIC_WRITE` → exclusive lock (`FOR UPDATE`)
- `PESSIMISTIC_FORCE_INCREMENT` → exclusive lock + bumps a version column

You typically also set a **timeout** to avoid indefinite blocking:

```java
Map<String, Object> props = Map.of("javax.persistence.lock.timeout", 5000);
em.find(Account.class, id, LockModeType.PESSIMISTIC_WRITE, props);
```

**In short:** at the DB level it's row locks acquired via `SELECT FOR UPDATE`/`FOR SHARE` and managed by the lock manager; in Java/JPA you trigger this by specifying a `LockModeType` on a find/query, which Hibernate translates into the appropriate locking SQL.

## The Classic Production Use Case: The Queue Pattern

Imagine 10 background worker instances trying to process tasks from a database table (e.g., an [[Outbox Pattern]] table):

- **Without `SKIP LOCKED`:** Worker 1 and Worker 2 both try to lock the first row. Worker 2 hangs until Worker 1 finishes, creating severe thread starvation and latency.
- **With `SKIP LOCKED`:** Worker 1 locks row 1. Worker 2 immediately skips row 1 and grabs row 2. All 10 workers work concurrently with zero blocking.

> [!warning] Failure modes & trade-offs
> - **Deadlocks:** If Transaction A locks Row 1 then Row 2, while Transaction B locks Row 2 then Row 1, the database aborts one of them. Prevent deadlocks by always acquiring locks in a **consistent deterministic order** (e.g., sorted by `id`).
> - **Connection pool exhaustion:** Holding a pessimistic lock across a slow external HTTP call keeps both the row lock *and* the DB connection tied up. Never hold DB locks across network I/O; use [[Timeout Pattern|lock timeouts]] (`SET LOCAL lock_timeout`) and keep [[Transaction|transactions]] short.
> - **Missing index = table lock:** In some engines (like MySQL InnoDB), `SELECT ... FOR UPDATE` on an unindexed predicate locks every scanned row/gap, effectively locking the entire table.

## Flashcards

What is the difference between `FOR UPDATE NOWAIT` and `FOR UPDATE SKIP LOCKED`?::`NOWAIT` fails immediately with an error if a target row is locked; `SKIP LOCKED` ignores locked rows and returns the next unlocked ones (ideal for DB-backed job queues).
^q-8hyy

How do you prevent deadlocks when transactions must pessimistically lock multiple rows?::Always acquire locks in a deterministic global order (for example, sorted ascending by primary key).
^q-j33r

Never hold a database pessimistic lock across external network calls; set a strict `lock_timeout` and keep transactions short. #card-reminder
^q-6j9u

## Related

- [[Optimistic Locking]]
- [[Pessimistic Concurrency Control]]
- [[Database Lock]]
- [[Transaction]]
- [[Outbox Pattern]]
- [[Distributed Lock]]
- [[JPA]]
- [[Hibernate]]