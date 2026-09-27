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