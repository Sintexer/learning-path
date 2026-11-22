---
flashcards:
  q-mihs: { nid: 1795356724693, hash: esizjsu9, sync: nup2rzb8 }
---

A **bitmap** is a data structure that represents a set of values using individual **bits** (0 or 1), where each bit indicates the presence, absence, or truth of some condition for a corresponding record or position. In the context of databases, a bitmap is most commonly encountered as a [[Bitmap Index]] — an indexing technique that uses bit arrays instead of trees or hashes to speed up queries, especially on columns with a small number of distinct values (low cardinality), and heavily used in [[Column-Oriented Storage]] and [[OLAP]] engines.

## Other Database Uses of Bitmaps

Beyond [[Bitmap Index|bitmap indexes]], bitmaps appear elsewhere in database internals:

- **Existence/visibility maps** — e.g., [[PostgreSQL]]'s *visibility map* uses one bit per page to track whether a page contains only tuples visible to all transactions, speeding up `VACUUM` and index-only scans (alongside [[MVCC]]).
- **[[Bloom Filter|Bloom filters]]** — a probabilistic bitmap-based structure used to quickly test whether a value is *definitely not* in a set, reducing unnecessary disk lookups.
- **Free space / allocation maps** — bitmaps track which disk blocks or pages are free or occupied.
- **Bitmap heap scans** — some query planners (e.g., [[PostgreSQL]]) build a temporary in-memory bitmap of matching row locations from multiple indexes, then bitwise `AND`/`OR` them together before fetching actual rows — combining benefits of several indexes in one query.

> [!warning] Failure modes & trade-offs
> - **High-cardinality blowup**: Uncompressed bitmaps scale with `(distinct values × row count)`. On high-cardinality columns (e.g., user IDs), raw bitmaps explode in memory unless compressed (e.g., Roaring Bitmaps or run-length encoding).
> - **Write contention**: Updating a single row in a bitmap index often locks an entire compressed bitmap chunk, making bitmap indexes poor for write-heavy [[OLTP]] workloads.

## Why are bitmap indexes well-suited to OLAP but poorly suited to high-concurrency OLTP? #card
- **[[OLAP]]**: Bitwise `AND`/`OR` operations on compressed bitmaps over low-cardinality columns are extremely fast and CPU-cache friendly.
- **[[OLTP]]**: Mutating a single row modifies compressed bitmap segments and causes heavy lock contention, degrading concurrent write throughput.

What does [[PostgreSQL]]'s visibility map track using one bit per page?::Whether all tuples on a heap page are visible to all active transactions, enabling index-only scans and skipping pages during `VACUUM`.
^q-mihs

## Related
- [[Bitmap Index]]
- [[Bloom Filter]]
- [[Column-Oriented Storage]]
- [[PostgreSQL]]
- [[OLAP]]
- [[Data Structures]]
