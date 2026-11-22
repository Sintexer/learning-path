---
flashcards:
  q-wky7: { nid: 1795356726299, hash: 5sqce3f6, sync: iwncg4h2 }
  q-s4we: { nid: 1795356726316, hash: 2pywq6rv, sync: vkrwpkm3 }
---

Database-Level Compare-And-Swap ([[Compare-And-Swap|CAS]]) — eliminates the [[Check-Then-Act]] race condition in a single atomic SQL statement.

Under the SQL standard and in engines like [[PostgreSQL]] and MySQL (InnoDB):

1. When an `UPDATE` executes, the storage engine acquires an **exclusive row-level write lock** (see [[Database Lock]]) on the target row before evaluating changes.
2. Even under the default isolation level (**Read Committed**), if another transaction was updating that row at the exact same millisecond, our transaction **waits** for that transaction to finish.
3. Once the lock is passed to our query, the engine **re-reads the latest committed version of the row**.
4. It evaluates the `WHERE` clause: `id = 100 AND balance >= 10`.
    - If the balance is still ≥10, it updates the row and decrements.
    - If the previous transaction reduced the balance to $5$, the condition evaluates to `false`. No update occurs.
5. The lock is released.

## The Pattern in Application Code

Instead of fetching first, you issue the write directly:

```sql
UPDATE accounts 
SET balance = balance - 10 
WHERE id = 100 AND balance >= 10;
```

In your application, you inspect the **Row Count (Rows Affected)** returned by the JDBC/database driver:

```java
int updatedRows = jdbcTemplate.update(
    "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
    amount, accountId, amount
);

if (updatedRows == 0) {
    throw new InsufficientFundsException("Balance too low or account missing");
}
// Proceed to order completion...
```

## When can you NOT use this pattern?

- If you need to calculate complex business logic in your application code that SQL cannot do (e.g., calling an external risk-scoring service or third-party fraud API based on the balance).
- If you must record an `orders` row containing data that depends on fields from multiple parent entities simultaneously. In that case, you must open a multi-statement transaction and use `SELECT ... FOR UPDATE` ([[Pessimistic Locking]]).

> [!tip] Production reality
> - **Prefer this over [[Optimistic Locking]] for hot rows:** On high-contention counters (inventory, wallet balance, quota), `UPDATE ... WHERE stock >= ?` serializes cleanly in the DB with zero client retries, whereas `@Version` optimistic locking triggers retry storms.
> - **Return new state in one round-trip:** In [[PostgreSQL]], append `RETURNING balance` to confirm the update and read the post-mutation value without a second query.
> - **Make retries safe:** If the client times out after the DB commits, retrying `balance = balance - 10` double-charges the user. Pair with an [[Idempotency Key]] stored in the same [[Transaction]].

## Flashcards

Why does `UPDATE accounts SET balance = balance - 10 WHERE id = 100 AND balance >= 10` prevent lost updates even under Read Committed?::The engine acquires an exclusive row lock, waits for any in-flight writer to commit, and re-evaluates the `WHERE` predicate against the latest committed row version.
^q-wky7

How does application code know whether an [[Atomic Conditional Mutation]] succeeded?::By checking that the affected row count returned by the database driver is `1` (or `> 0`); `0` means the row was missing or the guard condition failed.
^q-s4we

## Related

- [[Compare-And-Swap]]
- [[Check-Then-Act]]
- [[Optimistic Locking]]
- [[Pessimistic Locking]]
- [[Database Lock]]
- [[Idempotency Key]]
- [[Transaction]]