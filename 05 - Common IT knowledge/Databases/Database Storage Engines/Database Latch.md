In database systems, the term **latch** has a very specific, technical meaning that is distinct from a regular database "lock."

A **latch** is a **low-level, short-lived synchronization primitive (like a mutex, spinlock, or read/write lock) used by the database engine to protect internal, in-memory data structures from concurrent thread access.**

In everyday software engineering, people use "lock" and "latch" interchangeably. However, in database internals (PostgreSQL, MySQL/InnoDB, Oracle, SQL Server), they mean completely different things:

| Feature               | **Latch** (Lightweight Lock)                                                                                    | **Lock** (Database Transaction Lock)                                                                |
| :-------------------- | :-------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| **What it protects**  | **Physical memory data structures** (e.g., a specific B-Tree page currently loaded in RAM, hash table buckets). | **Logical database objects** (e.g., a row in a table, a whole table, a transaction).                |
| **Duration**          | **Microseconds to milliseconds.** Acquired, modified, and released immediately.                                 | **Seconds to minutes.** Held for the entire duration of a transaction until `COMMIT` or `ROLLBACK`. |
| **Who is it for?**    | The **database engine threads** (internal implementation detail, invisible to the user).                        | The **user / application** (used to guarantee ACID isolation levels).                               |
| **Rollback support?** | None. If a crash happens, you recover via logs.                                                                 | Full support. Releasing a lock rolls back transactional changes.                                    |
| **Example**           | *A C++ `pthread_rwlock` or mutex protecting a pointer to a child node in a B-Tree.*                             | *`SELECT ... FOR UPDATE` on row ID 42.*                                                             |

## Latch Crabbing

To traverse down a B-Tree safely without locking the entire tree, databases use an algorithm called **Latch Crabbing**:
1. Latch the **Root node** (Shared).
2. Find the child page ID.
3. Latch the **Child node** (Shared).
4. Release the latch on the **Root node**.
5. Repeat down the tree until you reach the leaf page.

> *(It is called "crabbing" because the thread holds two nodes at once as it steps down, moving like a crab).* 

While this prevents corruption, acquiring and releasing latches millions of times per second causes heavy CPU cache contention across multi-core processors.

### Shared latch

A **Shared Latch** (often called an **S-latch** or a **Read-Latch**) is a concurrency control primitive that allows **multiple threads to read the same data at the same time, but prevents any thread from modifying it.**

It is the database implementation of the classic computer science concept known as a **Reader-Writer Lock** (`RWLock`).

Latches generally come in two flavors:
1. **Shared (S-Latch):** _"I only want to read this memory page. I won't change anything."_
    - Multiple threads can hold an S-Latch on the same page simultaneously.
    - If 50 threads want to read Page 4 at the same time, they can all acquire an S-Latch together and read concurrently without waiting.
2. **Exclusive (X-Latch):** _"I want to modify this memory page. Nobody else can touch it."_
    - Only **one** thread can hold an X-Latch at a time.
    - It blocks **all other readers and writers**.