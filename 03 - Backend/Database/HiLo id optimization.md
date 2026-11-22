### Understanding the HiLo Identifier Generator

In database-driven applications, generating unique primary keys is a fundamental task. While many developers rely on database-native `AUTO_INCREMENT` columns, this approach often creates a performance bottleneck: every time you save an entity, the application must wait for a "round-trip" to the database to retrieve the newly generated ID.

The **HiLo (High-Low) algorithm** is a clever optimization designed to solve this by shifting ID generation from the database to the application memory.

#### How It Works
The HiLo algorithm splits the ID into two components:
1.  **High (Hi):** A value fetched from a dedicated database table or sequence. This acts as a "bucket" or "block" identifier.
2.  **Low (Lo):** A range of numbers (e.g., 0–99) that the application is authorized to use for that specific "High" bucket.

When the application starts or exhausts its current range, it performs a single transaction to increment the "High" value in the database. Once it has that value, it can generate hundreds of unique IDs locally in memory without further database communication. The formula is typically: `ID = (High * MaxLo) + Low`.

#### Why Use HiLo?
*   **Performance:** By reducing the number of database round-trips, you significantly decrease latency during bulk inserts (especially in [[Hibernate]]/[[JPA]], where `IDENTITY` columns disable JDBC batching).
*   **Predictability:** Unlike random UUIDv4s, HiLo IDs are numeric and sequential, which preserves [[B-Tree Index]] insertion locality and avoids page splits.
*   **Interoperability:** Because the "High" value is stored in a standard database table or sequence, multiple applications can share the same ID generation logic, provided they follow the same mathematical contract.

#### The Trade-off
The primary drawback of HiLo is the **"social contract."** If you have multiple applications (e.g., a Java service and a Python script) writing to the same database, they must all be configured to use the exact same HiLo logic and the same shared table. If one client deviates, you risk ID collisions.

While modern distributed systems often favor **UUIDv7** (time-ordered, see [[PostgreSQL]]) or **Snowflake IDs** to avoid this coordination, HiLo (and Hibernate's `pooled` / `pooled-lo` sequence optimizers) remains a classic, highly efficient pattern for monolithic or tightly coupled enterprise systems.

> [!tip] Production reality
> - **Gaps on crash:** If an application instance crashes after using only 3 IDs out of a 100-ID block, the remaining 97 IDs are permanently skipped. Never rely on HiLo IDs being gap-free for business invariants (e.g., legal invoice numbering).
> - **Prefer `pooled-lo` over legacy HiLo:** In modern [[Hibernate]] with [[PostgreSQL]] sequences (`INCREMENT BY 50`), the `pooled-lo` optimizer achieves the same zero-round-trip batching while letting non-Hibernate clients safely call `nextval()` on the same sequence without colliding.

## Flashcards

How does the HiLo ID generator eliminate per-insert database round-trips?::The app fetches one High bucket value from the DB and generates `MaxLo` sequential IDs locally in memory using `ID = (High * MaxLo) + Low`.

Why does [[Hibernate]] disable JDBC insert batching when using `GenerationType.IDENTITY`?::Because Hibernate must execute each `INSERT` immediately to obtain the database-generated primary key before flushing the persistence context.

What is the main operational risk of the classic HiLo algorithm in multi-writer databases?::The "social contract" — any external script or service inserting without following the exact same HiLo formula can collide with in-memory allocated IDs.

## Related

- [[Hibernate]]
- [[JPA]]
- [[B-Tree Index]]
- [[PostgreSQL]]