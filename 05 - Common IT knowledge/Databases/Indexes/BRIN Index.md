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

- **Footprint:** A B-Tree on a 1-billion-row table can easily take **30–50 GB** of RAM/Disk. A BRIN index on the same table can fit in **a few hundred Kilobytes**.
- **Connection to ClickHouse:** ClickHouse’s Primary Index works on a very similar principle (sparse indexing across granules of 8,192 rows storing min/max marks) rather than pointer-per-row dense indexing!