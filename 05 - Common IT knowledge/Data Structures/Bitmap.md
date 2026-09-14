A **bitmap** is a data structure that represents a set of values using individual **bits** (0 or 1), where each bit indicates the presence, absence, or truth of some condition for a corresponding record or position. In the context of databases, a bitmap is most commonly encountered as a [[Bitmap Index]] — an indexing technique that uses bit arrays instead of trees or hashes to speed up queries, especially on columns with a small number of distinct values.

## Other Database Uses of Bitmaps

Beyond bitmap indexes, bitmaps appear elsewhere in database internals:

- **Existence/visibility maps** — e.g., [[PostgreSQL]]'s _visibility map_ uses one bit per page to track whether a page contains only tuples visible to all transactions, speeding up vacuum and index-only scans.
- **[[Bloom filter]]s** — a probabilistic bitmap-based structure used to quickly test whether a value is _definitely not_ in a set, reducing unnecessary disk lookups.
- **Free space / allocation maps** — bitmaps track which disk blocks or pages are free or occupied.
- **Bitmap heap scans** — some query planners (e.g., PostgreSQL) build a temporary in-memory bitmap of matching row locations from multiple indexes, then AND/OR them together before fetching actual rows — combining benefits of several indexes in one query.
