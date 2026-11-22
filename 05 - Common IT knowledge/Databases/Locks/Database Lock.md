Different [[Database Storage Engines|storage engines]] implement row-level locking very differently. The two most common architectures are **In-Memory Lock Managers** (MySQL/InnoDB, SQL Server) and **Tuple-Header Locks** ([[PostgreSQL]]).

## Tuple-Header Lock

PostgreSQL avoids keeping a massive list of individual row locks in RAM. If an application locks 5,000,000 rows, an in-memory lock table could exhaust server memory (or trigger **lock escalation** to a full table lock, as in SQL Server).

Instead, Postgres stores the lock **directly inside the row itself (on the disk page)**:
- Every tuple (row) has a hidden header field: `xmax`.
- When a transaction runs `SELECT ... FOR UPDATE` or `UPDATE`, Postgres writes that transaction's ID into the row’s `xmax` field and flips internal status bits (called `infomasks`) to indicate an exclusive lock.
- If a second transaction arrives wanting to lock the same row, it inspects `xmax`, checks the PostgreSQL transaction status (`pg_xact`), sees that the transaction holding `xmax` is still active, and **sleeps (blocks)** waiting for that transaction ID to commit or rollback.

_(What if multiple transactions want a shared read lock? Postgres uses a special mechanism called **MultiXactId** to track an array of transaction IDs)._

## In-Memory Lock Table

InnoDB uses an internal in-memory data structure (the **Lock System**).
- It does not write lock flags to the disk page for reads.
- It allocates **Lock Objects** in RAM. To save memory, InnoDB does not create one lock object per row; it creates a lock object **per disk page**, containing a **[[Bitmap|bitmap]]** where each bit represents a specific row slot inside that page.
- **InnoDB also uses Gap Locks and Next-Key Locks:** When you lock rows in InnoDB, it locks not only the existing rows but also the "gaps" (empty spaces between keys in the [[B-Tree Index|B-Tree]]) to prevent other transactions from inserting new rows (phantoms) between your data.

> [!warning] Failure modes & trade-offs
> - **Tuple-header lock I/O cost:** In [[PostgreSQL]], even `SELECT ... FOR UPDATE` dirties the heap page and generates [[WAL]] records because the lock is written into `xmax`. Heavy locking on large sets can also cause `MultiXactId` wraparound pressure.
> - **Lock escalation vs. gap-lock deadlocks:** Engines with in-memory lock tables (like SQL Server) may escalate thousands of row locks into a table lock, stalling unrelated queries. In InnoDB, gap locks frequently cause unexpected deadlocks on concurrent `INSERT`s into non-overlapping keys.
> - **Locks vs. [[Database Latch|latches]]:** Locks are logical, held for the duration of a [[Transaction]], and support deadlock detection. [[Database Latch|Latches]] are physical, held for microseconds to protect in-memory data structures like B-Tree pages.

## Flashcards

How does [[PostgreSQL]] store row-level locks without running out of RAM when locking millions of rows?::Directly inside each row's tuple header (`xmax` + `infomask` bits) on the heap page, avoiding an in-memory per-row lock table.

In InnoDB, ==Gap Locks and Next-Key Locks== lock the empty spaces between [[B-Tree Index|B-Tree]] keys to prevent phantom inserts.

What is the key difference between a [[Database Lock]] and a [[Database Latch]]?::Locks protect logical database contents for the duration of a transaction; latches protect internal physical memory structures (like B-Tree pages) for microseconds.

## Related

- [[Pessimistic Locking]]
- [[Optimistic Locking]]
- [[Database Latch]]
- [[MVCC]]
- [[Transaction]]
- [[PostgreSQL]]
- [[Distributed Lock]]