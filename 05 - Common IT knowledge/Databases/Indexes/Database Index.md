Index is an *additional* data structure that is derived from the primary data. Many databases allow you to add and remove indexes, and this doesn't affect the contents of the database; it only affects the performance of queries. Maintaining additional structures incurs overhead, especially on writes. 

There are different kinds of index implementation:
- [[B-Tree Index]]
- [[Hash Index]]
- [[GIN Index]]
- [[GiST Index]]
- [[BRIN Index]]
- [[Bitmap Index]]

> [!info]
> An index is fundamentally an auxiliary data structure designed to minimize disk/buffer-pool I/O. Without an index, the engine must perform a **Sequential Scan (Full Table Scan)**, reading every single disk page allocated to the table.

Any kind of index usually trades write performance (as we have to update the index any time the data is written) for read performance. That is why databases don't usually index everything by default, but require you to choose indexes manually, using your knowledge of the application typical query patterns.

[[NoSQL database]]s usually use:
- [[#Log Structured Indexes]]
- [[#Hash Index]]

[[Relational Database]]s usually use:
- [[B-Tree Index]]

## Why indexes matter: use-cases

Normally, when you run:

```sql
SELECT * FROM users ORDER BY age;
```

If there is **no index** on `age`, the database must:

1. Fetch all rows from disk into memory (`work_mem`).
2. Run a sorting algorithm (like QuickSort or external MergeSort on disk if it exceeds RAM).
3. Emit the sorted rows.  In an `EXPLAIN ANALYZE` execution plan, this shows up as an expensive node called **`Sort`**.

A B-Tree leaf node is **already physically sorted** on disk, and leaf pages are tied together via a **doubly linked list**.

## Partial Index (Filtered Index)

An index built on **only a subset of rows** in a table, filtered by a `WHERE` condition — instead of indexing the entire table.

```sql
CREATE INDEX idx_active_users ON users (email)
WHERE status = 'active';
```

**Why use it:**

- Much smaller index → faster to scan, cheaper to maintain, less disk space.
- Great when queries almost always filter on a specific condition (e.g., only "active" rows, or `deleted_at IS NULL`), and the excluded rows are large in number but rarely queried.

The query's `WHERE` clause must match (or logically imply) the index's condition for the planner to use it.

```sql
SELECT email FROM users WHERE status = 'active' AND email = 'x@y.com';
-- ✅ can use idx_active_users

SELECT email FROM users WHERE status = 'inactive';
-- ❌ can't use it, condition doesn't match
```

**Common use cases:** soft-delete flags, boolean status columns, excluding NULLs, uniqueness constraints on a subset (e.g., "only one active session per user").

### Covering Index

An index that **includes all the columns a query needs**, so the database can answer the query **entirely from the index** without touching the actual table (a.k.a. an **index-only scan**).

```sql
CREATE INDEX idx_covering ON orders (customer_id) INCLUDE (order_date, total);
```

or, without `INCLUDE`, by putting extra columns directly in the index key:

```sql
CREATE INDEX idx_covering ON orders (customer_id, order_date, total);
```

**Why use it:**

- Avoids the extra "heap fetch" (jumping from the index entry to the actual table row) — a huge performance win for read-heavy queries.
- Best for queries like:

```sql
  SELECT order_date, total FROM orders WHERE customer_id = 42;
```

Here, `customer_id` is used for filtering, and `order_date`/`total` are just being read — an index-only scan satisfies it fully.

**`INCLUDE` vs. composite key columns:**

- Columns in the actual index **key** (before `INCLUDE`) are sorted and can be used for filtering/sorting.
- Columns in `INCLUDE` are just carried along as payload — not sorted, not usable for filtering, only for returning data — which keeps the index smaller/cheaper to maintain than making them full key columns.

**Trade-off:** covering indexes are larger (more data duplicated in the index) and slightly slower to update, in exchange for much faster reads.

PostgreSQL has an optimization feature called [[PostgreSQL#Visibility Map]].

### Combining Partial and Covering Indexes

The two are complementary and often used together:

```sql
CREATE INDEX idx_active_orders ON orders (customer_id) 
INCLUDE (order_date, total)
WHERE status = 'active';
```

This index is small (only active orders), and index-only (no heap fetch needed) — ideal for a hot, frequently-run query on a narrow slice of data.

## Log Structured Indexes

Log-structured indexes use [[Database Log]] at their core to maintain the records based on the write order.

- [[LSM Tree]] with [[SSTable]]

## Comparing B-Trees and LSM-Trees

Generally, B-Trees are faster for read, while LSM-Trees are faster for write. However numbers depend on the concrete workload and etc.

A B-tree index must write every piece of data at least twice: once to the write-ahead log, and once to the tree page itself (and perhaps again as pages are split). There is also overhead from having to write an entire page at a time, even if only a few bytes in that page changed. Some storage engines even overwrite the same page twice in order to avoid ending up with a partially updated page in the event of a power failure

1 Why LSM-Trees Win on Writes & Storage:
* **High Write Throughput:** LSM-trees turn random writes into sequential writes on disk, making them significantly faster at absorbing high-volume ingestion than B-trees (which must update random pages in place).
* **Better Compression & Less Waste:** B-trees suffer from **internal fragmentation** (half-empty disk pages created by page splits). LSM-trees don't use fixed-size pages and rewrite data cleanly during compaction, resulting in substantially smaller files on disk.

2 The Cost of LSM-Trees: Compaction & Write Amplification:
* **Write Amplification:** Because SSTables are repeatedly merged and rewritten over time, a single incoming write causes multiple physical disk writes. This eats into available disk bandwidth and accelerates SSD wear.
* **Tail Latency Spikes ($p99$):** Background compaction competes with active read/write queries for limited disk I/O. When disk bandwidth saturates, queries must wait, causing sudden latency spikes (making B-trees far more predictable).
* **Compaction Falling Behind:** If write volume exceeds compaction speed, unmerged SSTable files pile up. This simultaneously degrades read latency (more files to scan) and risks filling the disk completely.

3 Why B-Trees Still Dominate for Transactions:
* **Deterministic Location:** In a B-tree, a key exists in **exactly one place**. In an LSM-tree, multiple versions of a key are scattered across several SSTables.
* **Simpler Transaction Isolation:** Because a key lives in a single known node, databases can easily attach range locks directly to the tree structure, making strong ACID transactional semantics much easier to implement.

### The Bottom Line

* **Use LSM-Trees** when write throughput and storage efficiency are the primary bottlenecks.
* **Use B-Trees** when you need predictable read latency, rock-solid $p99$ response times, and strong transactional locks.

## Hash Index


## Other indexing structures

### Secondary Indexes

Indexes listed below are like a *primary key* index in the relational model. It is also very common to have secondary indexes to boost the joins performance.

A secondary index can easily be constructed from a key-value index. The main difference is that keys are not unique; i.e., there might be many rows (documents, vertices) with the same key. This can be solved in two ways: either by making each value in the index a list of matching row identifiers (like a postings list in a full-text index) or by making each key unique by appending a row identifier to it. Either way, both B-trees and log-structured indexes can be used as secondary indexes.

### Storing data within an index

When an index finds a matching key, it needs to provide the associated data. Databases use three distinct strategies to store this data:

#### 1. Heap Files (Non-Clustered / Reference-Based)
* **How it works:** The actual table rows live in an unordered storage pool called a **heap file**. The index stores only the key and a **pointer (memory/disk address)** to that heap file.
* **Benefit:** **Zero duplication.** If you have 5 different secondary indexes, they all point to the same single row in the heap. 
* **Drawbacks:**
  * **The "Extra Hop":** Reads are slower because the database must first read the index, then make a second I/O hop to fetch the row from the heap.
  * **Update penalties:** If an updated row grows larger than its original space, it must move to a new spot in the heap, requiring either updating *all* other secondary indexes or leaving behind a slower forwarding pointer.

#### 2. Clustered Indexes (Data Stored Directly in the Index)
* **How it works:** The actual row data is stored **directly inside the leaf nodes** of the primary index itself (e.g., MySQL InnoDB primary keys).
* **Benefit:** **Blazing fast reads** for primary key lookups—eliminates the "extra hop" entirely.
* **Drawback:** Secondary indexes must now point to the primary key (rather than a raw disk offset), requiring a two-stage traversal for secondary lookups.

#### 3. Covering Indexes (The Hybrid Compromise)
* **How it works:** A secondary index that stores the indexed key **plus a few extra frequently requested columns** (an "index with included columns").
* **Benefit:** Allows the database to fulfill specific queries entirely within the index itself, avoiding any trip to the main table or heap file (known as an **index-only scan**).

### Multi-column and Multi-dimensional indexes

Standard single-key indexes fall short when queries need to filter on multiple attributes simultaneously. The most common solution is a **concatenated index**, which joins multiple columns into a single ordered key (like `(lastname, firstname)` in a phone book). While efficient for queries matching the full key or just the leading column, it cannot help when searching solely for subsequent columns (such as looking up people by `firstname` alone). 

For true multi-dimensional range searches — such as bounding-box geospatial queries on `(latitude, longitude)` or filtering weather data by `(date, temperature)`— concatenated B-trees and LSM-trees are ineffective because they can only order data along one dimension at a time, forcing the engine to scan a wide range along the first axis and manually filter the rest. To narrow down across multiple dimensions at once, databases must either project multiple coordinates into a single dimension using space-filling curves or rely on specialized tree structures like **R-trees** (commonly used in [[PostGIS]]). This approach applies not only to geographic maps, but to any domain requiring simultaneous range filtering over continuous properties, such as searching e-commerce inventories by color ranges `(red, green, blue)` or querying multi-variable sensor logs.

### Fuzzy Querying and Full-Text Search

Traditional database indexes rely on exact matches or sorted range comparisons, making them incapable of handling typos, synonyms, or grammatical variations. To support this kind of fuzzy querying, full-text search engines like [[Apache Lucene]] use linguistic analysis and edit-distance algorithms (which measure character insertions, deletions, or substitutions) to find approximate matches. Under the hood, Lucene organizes its vocabulary in an SSTable-like term dictionary on disk, but instead of using a simple sparse key index like LevelDB, it uses an in-memory finite state automaton structured similarly to a [[Trie]]. This structure can be evaluated as a Levenshtein automaton, allowing the database to efficiently navigate character transitions and locate all terms within a specified edit distance without having to scan through every word in the dictionary. More advanced approaches to non-exact searching extend beyond character mechanics into document classification and machine learning.