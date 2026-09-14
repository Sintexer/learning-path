
[[Pessimistic Locking]] assumes conflicts are likely, so it locks data **before** modifying it, blocking other transactions from touching the same rows until the lock is released.

Optimistic locking assumes conflicts are **rare**, so it doesn't lock anything upfront. Instead, it lets transactions proceed freely and only checks for conflicts **at commit/update time**.

**Mechanics:**
- Each row has a **version marker** — typically a `version` integer column, or a `updated_at` timestamp/checksum.
- When you read a row, you also read its current version (e.g., `version = 5`).
- When you update it, the `UPDATE` statement includes a `WHERE` clause checking that version hasn't changed
- If another transaction already updated the row (and bumped the version to 6), this update affects **0 rows** — the app detects that and throws a conflict/exception, typically asking the user to retry.
- No locks are held during the "think time" between read and write, so it scales much better under low-contention, read-heavy workloads.

## Controlling It from Java (JPA/Hibernate)

**1. Add a `@Version` field to the entity:**

```java
@Entity
public class Account {
    @Id
    private Long id;

    private BigDecimal balance;

    @Version
    private int version;
}
```

Hibernate automatically:

- Includes `version` in the `WHERE` clause of every `UPDATE`/`DELETE`.
- Increments it on every successful update.
- Throws `OptimisticLockException` (wrapping JPA's `jakarta.persistence.OptimisticLockException`) if the affected row count is 0.

**2. Handling the conflict:**

```java
try {
    Account acc = em.find(Account.class, id);
    acc.setBalance(acc.getBalance().add(amount));
    em.merge(acc);
    em.flush();
} catch (OptimisticLockException e) {
    // retry, or return a "please try again" response to the user
}
```

**3. Explicit lock mode (rarely needed, since `@Version` handles it automatically):**

```java
em.find(Account.class, id, LockModeType.OPTIMISTIC);       // just verifies version at commit
em.find(Account.class, id, LockModeType.OPTIMISTIC_FORCE_INCREMENT); // forces a version bump even without field changes
```

**4. Spring Data JPA** — no special annotation needed; just having `@Version` on the entity is enough. Spring/Hibernate throws `ObjectOptimisticLockingFailureException` on conflict, which you catch and handle (often with a retry via `@Retryable`).

## Optimistic vs. Pessimistic — Quick Contrast

| |Optimistic|Pessimistic|
|---|---|---|
| Locking | None until write | Row locked from first read |
| Conflict detection | At commit (version mismatch) | Prevented upfront (blocking) |
| Best for | Low contention, high concurrency | High contention, critical sections |
| Failure mode | Exception → retry | Waiting / timeout / deadlock |

**In short:** at the DB level, optimistic locking is just a conditional `UPDATE ... WHERE version = X`; in JPA you get this for free by adding `@Version` to the entity, and you handle conflicts by catching `OptimisticLockException` and retrying.