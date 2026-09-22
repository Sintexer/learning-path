
**Check-Then-Act** (also known as **TOCTOU** — Time-Of-Check to Time-Of-Use) is a concurrency bug pattern where code **checks a condition**, then **acts on the assumption that the condition still holds** — but between the check and the act, another thread/process/transaction can change the state, invalidating the assumption.

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

### How to Fix It

The general principle: make the check-and-act a **single atomic operation**, using one of:

- **Database constraints** — `UNIQUE` constraints + `INSERT ... ON CONFLICT DO NOTHING` (Postgres) / `INSERT IGNORE` (MySQL) / `MERGE` — let the DB enforce atomicity instead of checking first.
- **Pessimistic locking** — `SELECT ... FOR UPDATE` to lock the row before checking, so no one else can act on it concurrently.
- **Optimistic locking** — use a `version` column; if the act fails because the version moved, retry.
- **Atomic operations/CAS (compare-and-swap)** — e.g., `AtomicInteger.compareAndSet()` in Java, or `UPDATE ... SET balance = balance - x WHERE balance >= x` (folding the check into the `WHERE` clause of the act itself).
- **Concurrent collections** — `ConcurrentHashMap.putIfAbsent()` instead of manual `containsKey` + `put`.
- **Filesystem** — use atomic APIs (`Files.createFile()` which throws if it exists) instead of `exists()` + `create()`.
