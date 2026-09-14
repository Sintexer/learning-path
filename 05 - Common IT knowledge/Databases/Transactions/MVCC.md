Multi-Version Concurrency Control

In older database systems, if Transaction A was **reading** a table, it locked the table, and Transaction B had to **wait** before it could update a row. If Transaction B was **writing**, readers were blocked. Concurrency was terrible.

**MVCC solves this with a golden rule:**

> _"Readers never block writers; writers never block readers."_

## How MVCC physically works:

When you execute an `UPDATE` in [[PostgreSQL]], the engine **does not overwrite the old data on disk.**

Instead:

1. It writes a **brand new row version (tuple)** to an open disk page.
2. It leaves the old row version sitting right where it was, but marks it as obsolete.

Every row has hidden system headers:

- `xmin`: The transaction ID that **created** this row version.
- `xmax`: The transaction ID that **deleted or replaced** this row version.

```
Initial State (Row inserted by Transaction 100):
[ TID (1,1) | xmin: 100 | xmax: 0    | name: "Alice" ]

Transaction 105 runs: UPDATE users SET name = 'Bob' WHERE name = 'Alice';
Disk layout now:
[ TID (1,1) | xmin: 100 | xmax: 105  | name: "Alice" ] <-- marked as killed by 105
[ TID (1,2) | xmin: 105 | xmax: 0    | name: "Bob"   ] <-- new active version
```

If an older, long-running Transaction 102 queries `Alice`, it looks at `xmax: 105`, realizes 105 committed _after_ it started, and safely reads the old data!

**The downside:** These dead rows accumulate on disk. This is called **bloat**.

**The solution:** A background process called [[PostgreSQL#VACUUM]] cleans up obsolete rows whose `xmax` is older than all currently running transactions.