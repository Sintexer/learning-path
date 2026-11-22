---
flashcards:
  q-vpg3: { nid: 1795356731192, hash: zaf97mnv, sync: nxbey85u }
  q-9j4i: { nid: 1795356731219, hash: ard6efip, sync: yuyxeeqs }
---

Hibernate is a [[02 - Java/JPA/JPA|JPA]] implementation. But Hibernate existed before JPA. So it has many performance optimized features not present in JPA specification.

**non-JPA compliant features:**
- **HiLo**: extended identifier generators, implementing a [[HiLo id optimization|HiLo]] optimizer that’s interoperable with other database clients 
* **Transparent prepared statement batching:** Automatically groups multiple `INSERT`, `UPDATE`, or `DELETE` statements into a single database network trip to improve performance.
* **Customizable CRUD (@SQLInsert, etc.):** Allows you to override Hibernate’s default generated SQL with your own custom SQL or stored procedure calls for specific operations.
* **Static/Dynamic collection filters (@Filter):** Enables you to apply runtime criteria (like `tenant_id` or `active=true`) to collections or queries globally without manually adding them to every query.
* **Entity filters (@Where):** A static filter applied to an entity class that automatically appends a `WHERE` clause to every SQL query involving that entity.
* **Mapping to SQL fragments (@Formula):** Allows you to map a property to a calculated SQL expression (e.g., `(select count(*) from orders o where o.user_id = id)`) rather than a simple table column.
* **Immutable entities (@Immutable):** Tells Hibernate that an entity never changes, allowing it to skip "dirty checking" and improve performance.
* **More flush modes:** Provides finer control over *when* Hibernate synchronizes the in-memory state with the database (e.g., `MANUAL` prevents automatic flushing).
* **Querying 2nd-level cache by natural key:** Allows you to fetch entities directly from the cache using a business key (like an email address) instead of the primary key.
* **Entity-level cache concurrency strategies:** Defines how the cache handles concurrent access (e.g., `READ_WRITE` ensures data consistency in multi-user environments).
* **Versioned bulk updates through HQL:** Allows you to perform `UPDATE` or `DELETE` queries on many rows at once while automatically incrementing the `@Version` field for [[Optimistic Locking|optimistic locking]].
* **Exclude fields from optimistic locking:** Lets you specify that changes to certain "unimportant" fields should not trigger an `OptimisticLockException`.
* **Version-less optimistic locking:** Allows [[Optimistic Locking|optimistic locking]] even on tables that lack a `@Version` column by comparing the state of all entity fields (or only dirty ones) during an update.
* **Skipping pessimistic lock requests:** Allows a query to use `SKIP LOCKED` ([[Pessimistic Locking]]), which tells the [[Database|database]] to ignore rows already locked by other [[Transaction|transactions]] instead of waiting for them to be released (useful for DB-backed job queues / [[Outbox Pattern]] workers).

> [!warning] Failure modes & trade-offs
> - **`GenerationType.IDENTITY` disables JDBC batching:** Hibernate must know the primary key immediately on `persist()` to register the entity in the Persistence Context, so `IDENTITY` forces an immediate `INSERT` per row. Use `SEQUENCE` + pooled/[[HiLo id optimization]] to keep transparent JDBC batching working.
> - **2nd-level cache coherence:** Hibernate's L2 cache only sees writes going through Hibernate in that cluster. External SQL scripts, other microservices, or bulk native queries bypass it and leave stale cached entities.
> - **Dirty-checking overhead on large read queries:** Loading thousands of managed entities bloats the 1st-level cache and slows flush dirty-checking; use `@Immutable`, `readOnly = true` transactions, or DTO projections for read-heavy paths.

## Flashcards

Why does `GenerationType.IDENTITY` silently disable Hibernate JDBC `INSERT` batching?::Hibernate must obtain the generated primary key immediately on `persist()` to track the entity in the Persistence Context, forcing an immediate per-row `INSERT` instead of batching; use `SEQUENCE` with a pooled/[[HiLo id optimization|HiLo]] optimizer instead.
^q-vpg3

What is the practical use of Hibernate's `SKIP LOCKED` support in [[Pessimistic Locking]]?::Allows concurrent worker threads/nodes (e.g. [[Outbox Pattern]] or job-queue consumers) to claim unlocked rows immediately without blocking on rows already locked by other transactions.
^q-9j4i

## Related
- [[JPA]]
- [[HiLo id optimization]]
- [[Optimistic Locking]]
- [[Pessimistic Locking]]
- [[persistence]]
- [[Outbox Pattern]]
- [[Impedance mismatch]]