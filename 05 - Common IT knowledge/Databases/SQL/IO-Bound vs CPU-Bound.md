The distinction is based on **what resource limits the query’s performance**.

### I/O-bound query

An **I/O-bound query** spends most of its time waiting for data to be read from or written to storage.

Common causes:

- Large table or index scans
- Data not already available in PostgreSQL/shared buffers or the OS cache
- Slow storage
- Sorting or hashing that spills to temporary disk
- Bulk inserts, updates, or maintenance operations

Example:

```sql
SELECT *
FROM orders
WHERE description LIKE '%damaged%';
```

Without a suitable index, PostgreSQL may scan a large table from storage.

### CPU-bound query

A **CPU-bound query** spends most of its time performing computations after the data is available.

Common causes:

- Expensive joins or aggregations
- Sorting a large in-memory data set
- Complex expressions or functions
- JSON processing
- Regex matching
- PostGIS calculations
- Hashing, encryption, or compression
- Processing many rows even when they are cached

## Side-by-side comparison

|Aspect|I/O-bound|CPU-bound|
|---|---|---|
|Main bottleneck|Reading/writing storage|Computation|
|CPU utilization|Often low or waiting|Usually high|
|Disk activity|Usually high|Usually low to moderate|
|Warm cache effect|Often much faster|Often little difference|
|Common operations|Scans and disk spills|Joins, sorts, aggregates, functions|
|Main optimization|Access less data or improve I/O|Perform less computation|

## Important nuance

A query can be **both** I/O-bound and CPU-bound at different stages. For example:

1. A sequential scan reads millions of rows from storage—**I/O-bound**.
2. A hash join processes those rows—**CPU-bound**.
3. A large sort exceeds `work_mem` and spills—**I/O-bound again**.

The bottleneck can also change after caching. The first execution may be I/O-bound, while later executions become CPU-bound because the data is cached.

> [!tip] Production reality
> - **Diagnose before optimizing:** In [[PostgreSQL]], run `EXPLAIN (ANALYZE, BUFFERS)` — high `shared read` / `temp read/written` signals an **I/O-bound** query (add a [[Database Index]], increase `work_mem`, or leverage async I/O), while high execution time with mostly `shared hit` signals a **CPU-bound** query (simplify expressions, reduce row count earlier, or parallelize).
> - **Fix the right bottleneck:** Upgrading to faster NVMe storage or enabling [[PostgreSQL]] 18 async I/O will not speed up a CPU-bound regex or JSON parsing query.

## Flashcards

How can you tell from [[PostgreSQL]] `EXPLAIN (ANALYZE, BUFFERS)` whether a slow query is I/O-bound or CPU-bound?::High `shared read` or `temp read/written` blocks indicate I/O-bound storage waits; high execution time with almost all `shared hit` blocks indicates CPU-bound computation.

Adding faster disks or async I/O speeds up ==I/O-bound== queries, while reducing scanned rows, complex regexes, or expensive joins speeds up ==CPU-bound== queries.

## Related

- [[PostgreSQL]]
- [[Database Index]]
- [[Database Index Practical Usage]]
- [[Grouping]]
- [[OLTP and OLAP]]