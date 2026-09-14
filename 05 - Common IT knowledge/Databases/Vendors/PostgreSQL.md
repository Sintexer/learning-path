
## Visibility Map

> _In Postgres, how does the engine know if a row version in an index is visible to our transaction without checking the heap `xmin`/`xmax`?_  

PostgreSQL maintains a tiny bitmap called the **Visibility Map**.  
If a bit is set for a heap page, it guarantees that **all rows on that page are visible to all active transactions** (no uncommitted or dead tuples).  
An Index-Only Scan checks the Visibility Map:

- If the bit is `1` → It skips the heap read entirely.
- If the bit is `0` → It is forced to visit the heap to check [[MVCC]] visibility. (This is why running [[#VACUUM]] makes Index-Only Scans fast!)

## TOAST

The Oversized-Attribute Storage Technique.

PostgreSQL does not allow a single row to span across multiple heap pages. Every row header must fit inside one page. To make this work:

1. **Threshold check:** If a row exceeds ≈2 KB (a quarter of a page), the TOAST mechanism intervenes.
2. **Compression:** The engine first attempts to compress large columns (using `pglz` or `lz4`).
3. **Out-of-line storage:** If the data is still too big, the engine takes the large text/JSON value and cuts it into pieces (chunks) stored in a **separate, hidden physical table** called the **TOAST Table**.
4. **Pointer replacement:** In the primary table page, the actual text/JSON is replaced by a tiny **24-byte pointer** (called an On-Disk Toast Pointer) that points to the chunks in the TOAST table.

```
Main Table Page (8KB):
[ Row 1 | ID: 101 | Name: 'Doc' | TOAST Pointer: 0x93FA2 ]

TOAST Table Pages:
[ Chunk 1 (2KB) of 0x93FA2 ]
[ Chunk 2 (2KB) of 0x93FA2 ]
[ Chunk 3 (1KB) of 0x93FA2 ]
```

_Why does this matter in interviews?_  
If you run `SELECT id, name FROM users;`, Postgres **never reads the TOAST table**. It reads the heap pages fast because the large text payload isn't in the way.  
If you run `SELECT * FROM users;`, Postgres has to do random disk I/O to look up every chunk in the TOAST table to stitch the large JSON/text back together, which degrades performance. This is why senior engineers strongly advise against `SELECT *`.

## VACUUM

Related to [[MVCC]].

A background process called **`VACUUM`** cleans up obsolete rows whose `xmax` is older than all currently running transactions.

**Plain `VACUUM`:**
- Scans pages, marks dead tuple slots as "free space" inside the 8KB page.
- **Crucial:** It **does not return space to the OS**. It only makes space available for _future inserts_ in that same table.
- It runs concurrently in the background (**`autovacuum`**) and **does not block reads or writes**.

**`VACUUM FULL`:**
- Rewrites the entire table to a brand new disk file, compacting everything and returning disk space to the OS.
- **The Trap:** It acquires an **`ACCESS EXCLUSIVE` lock**. It blocks **ALL** reads and writes until it finishes. If run on a 500GB table in production, you will take down the entire system for hours! (Production alternatives: `pg_repack`).

## HOT (Heap-Only Tuples) Updates

Every time an update creates a new row, does it also have to insert a new entry into every single index on that table?  
If yes, write amplification would destroy performance!

PostgreSQL uses an optimization called **HOT Updates**:  
If you update a row:

1. The new row version fits inside the **exact same 8KB disk page** as the old version, **AND**
2. **None of the indexed columns were modified**.

Then, Postgres does **not** touch the indexes at all! The old tuple on that page acts as a pointer jumping directly to the new tuple. The indexes still point to the old TID, saving massive disk I/O.

## Special Types

### JSON

PostgreSQL supports two JSON types:

- `json`: Stored as raw text. Must be re-parsed on every query. Rarely used.
- `jsonb`: Stored as decomposed binary. Slower to insert, but fast to process and **indexable**.

#### Table Creation

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    profile JSONB
);
```

#### Creating the GIN Index

There are two ways to index JSONB with GIN:

**Option A: Default (`jsonb_ops`)** — Indexes every key and every value.
```sql
CREATE INDEX idx_users_profile ON users USING gin (profile);
```

**Option B: Specialized (`jsonb_path_ops`)** — Hashes the full path + value together. **Smaller and faster**, but only supports the `@>` (containment) operator.
```sql
CREATE INDEX idx_users_profile_fast ON users USING gin (profile jsonb_path_ops);
```

#### Querying using Index-Supported Operators

```sql
-- 1. Containment check (Uses the GIN index)
-- Finds users where profile has {"role": "admin"}
SELECT * FROM users WHERE profile @> '{"role": "admin"}';

-- 2. Check if a key exists (Uses default jsonb_ops index)
-- Finds users who have a "theme" key defined
SELECT * FROM users WHERE profile ? 'theme';

-- 3. Extracting a value and doing scalar comparison
-- NOTE: Plain GIN does NOT speed up this operator ("->>"):
SELECT * FROM users WHERE profile->>'role' = 'admin'; 
-- (To speed up this exact query, you would use a standard B-Tree expression index instead: 
-- CREATE INDEX idx_role ON users ((profile->>'role')); )
```

### Range Types

PostgreSQL has native data types called **Range Types**.

- `daterange`: A range of dates, e.g., `[2023-01-01, 2023-01-15]`
- `tsrange`: A range of timestamps, e.g., `[2023-01-01 10:00:00, 2023-01-01 12:00:00]`

Imagine booking meeting rooms or hotel rooms. In standard databases, you do:

```sql
CREATE TABLE bookings (
    room_id INT,
    start_time TIMESTAMP,
    end_time TIMESTAMP
);
```

If someone wants to book from `10:30` to `11:30`, checking for overlaps requires a messy query:  
`WHERE NOT (end_time <= '10:30' OR start_time >= '11:30')`  
A B-Tree cannot index `start_time` and `end_time` together in a way that handles this overlapping range query efficiently.

A [[GiST Index]] can index these ranges natively using bounding intervals, evaluating overlap queries in $O(log⁡N)$ time.

```sql
CREATE TABLE bookings (
    room_id INT,
    during TSRANGE
);

-- Check if a new slot overlaps with ANY existing booking using the "&&" operator:
SELECT * FROM bookings WHERE during && tsrange('2023-10-01 10:30', '2023-10-01 11:30');
```