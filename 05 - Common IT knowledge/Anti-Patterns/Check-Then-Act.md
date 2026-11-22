---
flashcards:
  q-ktn4: { nid: 1795356724921, hash: f88p7h3v, sync: gtpmgn4m }
  q-jtfg: { nid: 1795356724942, hash: d6xdxbyt, sync: fpb6mgbb }
  q-2j7y: { nid: 1795356724967, hash: wtf96jrd, sync: rx4xe6na }
---

**Check-Then-Act** (also known as **TOCTOU** — Time-Of-Check to Time-Of-Use) is a concurrency bug pattern where code **checks a condition**, then **acts on the assumption that the condition still holds** — but between the check and the act, another thread, process, or [[Transaction]] can change the state, invalidating the assumption.

### Generic Shape

```java
if (!map.containsKey(key)) {   // CHECK
    map.put(key, value);        // ACT
}
```

Between the `containsKey` check and the `put`, another thread could insert the same key. Result: a lost update, duplicate insert, or overwritten data — a classic **race condition**.

### Common Real-World Examples

**1. In-memory / application code:**

```java
if (account.getBalance() >= amount) {      // CHECK
    account.withdraw(amount);               // ACT — balance could've changed by now
}
```

**2. Database — "check then insert":**

```sql
SELECT COUNT(*) FROM users WHERE email = 'x@y.com';  -- CHECK
-- if 0, then:
INSERT INTO users (email) VALUES ('x@y.com');          -- ACT
```

Two concurrent transactions can both pass the check before either inserts → duplicate rows (unless a unique constraint catches it).

**3. Filesystem:**

```java
if (file.exists()) {        // CHECK
    file.delete();          // ACT — file could be gone or replaced by then
}
```

### Why It's Dangerous

- It's **invisible in single-threaded testing** — only shows up under real concurrency/load, making it a notoriously hard bug to reproduce and debug.
- It silently corrupts data rather than throwing an obvious error.

> [!warning] Failure modes & trade-offs
> Under [[Snapshot Isolation]] (like PostgreSQL's `REPEATABLE READ`), two concurrent transactions can both pass a multi-row check and write disjoint rows without triggering a write-write conflict — a subtle Check-Then-Act variant known as [[Write Skew]]. Row locks (`SELECT ... FOR UPDATE`), unique constraints, or `SERIALIZABLE` isolation are required to close it.

### How to Fix It

The general principle: make the check-and-act a **single atomic operation**, using one of:

- **Database constraints** — `UNIQUE` constraints + `INSERT ... ON CONFLICT DO NOTHING` ([[PostgreSQL]]) / `INSERT IGNORE` (MySQL) / `MERGE` — let the DB enforce atomicity instead of checking first.
- **[[Pessimistic Locking]]** — `SELECT ... FOR UPDATE` to lock the row before checking, so no one else can act on it concurrently.
- **[[Optimistic Locking]]** — use a `version` column ([[Optimistic Concurrency Control]]); if the act fails because the version moved, retry.
- **[[Atomic Conditional Mutation]] / [[Compare-And-Swap|CAS]]** — e.g., `AtomicInteger.compareAndSet()` in Java, or `UPDATE ... SET balance = balance - x WHERE balance >= x` (folding the check into the `WHERE` clause of the act itself).
- **Concurrent collections** — `ConcurrentHashMap.putIfAbsent()` instead of manual `containsKey` + `put`.
- **Filesystem** — use atomic APIs (`Files.createFile()` which throws if it exists) instead of `exists()` + `create()`.

Never validate a precondition in application code and assume it still holds during the subsequent write; push the invariant into an atomic primitive or database constraint. #card-reminder
^q-ktn4

Check-Then-Act (TOCTOU) race condition:::A concurrency bug where state changes between checking a condition and acting on it, causing lost updates, duplicates, or invariant violations.
^q-jtfg

## How do you eliminate a database Check-Then-Act race without heavy locking? #card
1. Enforce the invariant with a `UNIQUE` constraint and `INSERT ... ON CONFLICT` / `MERGE`.
2. Fold the check into the write via an [[Atomic Conditional Mutation]] (`UPDATE ... WHERE balance >= x`).
3. Use [[Optimistic Locking]] with a monotonic `version` column and retry on conflict.
^q-2j7y

## Related
- [[Atomic Conditional Mutation]]
- [[Compare-And-Swap]]
- [[Optimistic Concurrency Control]]
- [[Pessimistic Locking]]
- [[Critical Section]]
- [[Write Skew]]
