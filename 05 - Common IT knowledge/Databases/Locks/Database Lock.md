Different storage engines implement row-level locking very differently. The two most common architectures are **In-Memory Lock Managers** (MySQL/[[InnoDB]], SQL Server) and **Tuple-Header Locks** ([[PostgreSQL]]).

## Tuple-Header Lock

PostgreSQL avoids keeping a massive list of individual row locks in RAM. If an application locks 5,000,000 rows, an in-memory lock table could exhaust server memory.

Instead, Postgres stores the lock **directly inside the row itself (on the disk page)**:
- Every tuple (row) has a hidden header field: `xmax`.
- When a transaction runs `SELECT ... FOR UPDATE` or `UPDATE`, Postgres writes that transaction's ID into the row’s `xmax` field and flips internal status bits (called `infomasks`) to indicate an exclusive lock.
- If a second transaction arrives wanting to lock the same row, it inspects `xmax`, checks the PostgreSQL transaction status (`pg_xact`), sees that the transaction holding `xmax` is still active, and **sleeps (blocks)** waiting for that transaction ID to commit or rollback.

_(What if multiple transactions want a shared read lock? Postgres uses a special mechanism called **MultiXactId** to track an array of transaction IDs)._

## In-Memory Lock Table

InnoDB uses an internal in-memory data structure (the **Lock System**).
- It does not write lock flags to the disk page for reads.
- It allocates **Lock Objects** in RAM. To save memory, InnoDB does not create one lock object per row; it creates a lock object **per disk page**, containing a **bitmap** where each bit represents a specific row slot inside that page.
- **InnoDB also uses Gap Locks and Next-Key Locks:** When you lock rows in InnoDB, it locks not only the existing rows but also the "gaps" (empty spaces between keys in the B-Tree) to prevent other transactions from inserting new rows (phantoms) between your data.