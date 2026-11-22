## Block Range Index

BRIN does **not** index individual rows. Instead, it records summaries for contiguous physical disk pages (by default, one summary per **128 pages** = ~1MB of data).  

> [!important]
> Data must be sorted on disk for the BRIN

For each range of pages, BRIN stores only two values:

- **Minimum value**
- **Maximum value**

```
Physical Storage (Heap):
[ Page 0 - 127 ]: IDs 1 to 10,000      --> BRIN Entry: Min = 1, Max = 10,000
[ Page 128 - 255 ]: IDs 10,001 to 20,000 --> BRIN Entry: Min = 10,001, Max = 20,000
```

When you query: `WHERE id = 15400`, the engine consults the BRIN index:

1. Does it fall in `[1, 10000]`? No → Skip 128 pages entirely.
2. Does it fall in `[10001, 20000]`? Yes → Read those 128 pages sequentially and filter.

## The Strict Prerequisite

The data **must be physically sorted (correlated)** on disk along the indexed column.

- Works well for: `created_at` in append-only log tables, auto-incrementing serial IDs, time-series data.
- Completely fails for: Random UUIDs, unclustered updates, or frequently shuffled data.

## Why this matters (especially for ClickHouse/Analytical domains):

- **Footprint:** A [[B-Tree Index|B-Tree]] on a 1-billion-row table can easily take **30–50 GB** of RAM/Disk. A BRIN index on the same table can fit in **a few hundred Kilobytes**.
- **Connection to ClickHouse:** ClickHouse’s Primary Index works on a very similar principle (sparse indexing across granules of 8,192 rows storing min/max marks) rather than pointer-per-row dense indexing!

> [!warning] Failure modes & trade-offs
> - **Correlation decay from `UPDATE`/`DELETE`:** In [[PostgreSQL]], updating old rows writes new tuples into free space near the end of the heap, widening each block range's `[min, max]` interval until every block overlaps and BRIN degrades to a full sequential scan.
> - **Unsummarized tail pages:** Newly inserted pages are not summarized until `VACUUM` or `brin_summarize_new_values()` runs (unless `autosummarize = on` is set), so queries always scan the unsummarized tail.
> - **Tuning `pages_per_range`:** Default is 128 pages (~1 MB). Smaller values (e.g., 16–32) improve pruning precision at a slight index size increase.

## Flashcards

What does a [[BRIN Index]] store for each range of physical disk pages?::Only summary metadata — primarily the **minimum** and **maximum** column values across that contiguous range of pages (default 128 pages in [[PostgreSQL]]).

A [[BRIN Index]] is only effective when the column values are ==physically correlated (sorted) on disk==, such as `created_at` in an append-only table.

Why does a BRIN index degrade on a table with frequent historical `UPDATE`s?::Updates relocate tuples to different heap pages, stretching each block range's `[min, max]` bounds until the index can no longer skip pages.

## Related

- [[Database Index]]
- [[B-Tree Index]]
- [[Database Index Practical Usage]]
- [[PostgreSQL]]
- [[OLAP]]