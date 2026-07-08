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
* **Versioned bulk updates through HQL:** Allows you to perform `UPDATE` or `DELETE` queries on many rows at once while automatically incrementing the `@Version` field for optimistic locking.
* **Exclude fields from optimistic locking:** Lets you specify that changes to certain "unimportant" fields should not trigger an `OptimisticLockException`.
* **Version-less optimistic locking:** Allows optimistic locking even on tables that lack a `@Version` column by comparing the state of all entity fields (or only dirty ones) during an update.
* **Skipping pessimistic lock requests:** Allows a query to use `SKIP LOCKED`, which tells the database to ignore rows already locked by other transactions instead of waiting for them to be released.