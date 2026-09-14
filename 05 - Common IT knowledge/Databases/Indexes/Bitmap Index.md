An index relying on [[Bitmap]] data structure. Imagine a table with `N` rows and a column that can only take a handful of distinct values (e.g., `Gender`, `Status`, `Country`, `IsActive`). For each distinct value, a bitmap index creates a **bit vector of length N**:

- Bit `i` is `1` if row `i` has that value.
- Bit `i` is `0` otherwise.

### Example

|Row|Status|
|---|---|
|1|Active|
|2|Inactive|
|3|Active|
|4|Pending|
|5|Inactive|

This produces three bitmaps, one per distinct value:

```
Active:   1 0 1 0 0
Inactive: 0 1 0 0 1
Pending:  0 0 0 1 0
```

To find all rows where `Status = 'Active' OR Status = 'Pending'`, the database simply computes a bitwise **OR** between the two bitmaps:

```
Active:   1 0 1 0 0
Pending:  0 0 0 1 0
OR    =>  1 0 1 1 0
```

The resulting bitmap directly identifies rows 1, 3, and 4.

## Why Use Bitmaps in a Database?

1. **Extremely fast set operations** — AND, OR, XOR, and NOT on bitmaps map directly to CPU-level bitwise instructions, making them far faster than comparing values row by row.
2. **Compact storage for low-cardinality data** — a column with only a few distinct values (e.g., booleans, enums, categories) can be represented very efficiently as bits rather than full index entries.
3. **Efficient multi-condition queries** — combining filters on several low-cardinality columns (`WHERE gender = 'F' AND status = 'active' AND region = 'EU'`) becomes a matter of ANDing several bitmaps together, rather than performing multiple tree lookups and merging row ID lists.
4. **Good for read-heavy, analytical workloads** — bitmap indexes shine in data warehouses and [[OLAP]] systems where queries scan and filter large volumes of data, and updates are relatively infrequent.

## Bitmap Index vs. B-Tree Index

Related to [[B-Tree Index]].

| Aspect           | Bitmap Index                                                          | B-Tree Index                                               |
| ---------------- | --------------------------------------------------------------------- | ---------------------------------------------------------- |
| Best for         | Low-cardinality columns (few distinct values)                         | High-cardinality columns (many distinct values, e.g., IDs) |
| Storage          | Very compact for few distinct values                                  | Grows with number of distinct keys                         |
| Query type       | Combining multiple filters (AND/OR), COUNT, existence checks          | Range queries, equality lookups, sorting                   |
| Update cost      | Expensive — updating one row can require rewriting large bit segments | Cheap — localized tree updates                             |
| Concurrency      | Poor for write-heavy [[OLTP]] (row-level locking is harder)           | Good for OLTP workloads                                    |
| Typical use case | Data warehousing, OLAP, analytics                                     | Transactional systems, OLTP                                |

## Compression

Because bitmaps for low-cardinality columns are often long runs of the same bit, they compress extremely well. Common compression schemes include:

- **Run-Length Encoding (RLE)** — stores runs of repeated bits instead of every bit individually.
- **Byte-aligned Bitmap Code (BBC)**
- **Word-Aligned Hybrid (WAH)**
- **Roaring Bitmaps** — a modern, widely used compressed bitmap format that adapts its internal representation (arrays, bitmaps, or runs) depending on data density, offering both compact storage and fast operations. Roaring bitmaps are used in systems like [[Apache Lucene]], [[Apache Spark]], [[ClickHouse]], [[Druid]], and [[Elasticsearch]].

## Advantages

- Very fast logical operations (AND, OR, NOT, XOR).
- Space-efficient for low-cardinality data, especially when compressed.
- Naturally supports combining multiple predicates.
- Simple conceptual model — a matrix of rows × distinct values.

## Disadvantages

- Inefficient for high-cardinality columns (would need one bitmap per distinct value, exploding storage).
- Costly to update: inserting or deleting rows can require shifting or rewriting large bit arrays.
- Not ideal for highly concurrent write workloads due to locking granularity.
- Typically unordered — not naturally suited for range queries the way B-trees are.

## Where It's Used in Practice

- **[[Oracle Database]]** — offers native `BITMAP` index type, popular in data warehouse schemas (star schemas).
- **PostgreSQL** — does not have persistent bitmap indexes, but uses dynamic **bitmap index scans** internally to combine multiple B-tree/hash indexes for a single query.
- **Apache Druid, ClickHouse, Elasticsearch, Apache Spark** — use Roaring Bitmaps for indexing and fast filtering over large analytical datasets.
- **[[Data Warehouse]]s / OLAP cubes** — bitmap indexing is a foundational technique for star-schema fact/dimension filtering.
