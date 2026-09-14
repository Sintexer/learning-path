## Generalized Inverted Index

Think of GIN as the index at the back of a textbook.  
Instead of mapping a row to its contents (`RowID -> Column Value`), it breaks composite column values into **individual elements (lexemes, keys, array items)** and maps each element to a list of row locations where it appears (a **Posting List** or **Posting Tree**).

```
Array in Row 1: ['python', 'postgres']
Array in Row 2: ['postgres', 'go']

GIN Internal Mapping:
'go'       -> [Row 2]
'postgres' -> [Row 1, Row 2]
'python'   -> [Row 1]
```

A standard [[B-Tree Index]] indexes a **whole cell value**. It maps:   `Row -> Value`

A GIN index unpacks a cell containing multiple items and maps:   `Internal Item -> List of Rows containing it`

## Core Use Cases:

1. **Full-Text Search (FTS):** Searching documents via `tsvector` and `tsquery`.
2. **Arrays:** Finding rows where an array contains an item (`tags @> ARRAY['backend']`).
3. **JSONB:** Querying deep documents (`data @> '{"role": "admin"}'`).

## The Interview Trade-off:

- **Lookup:** Blazing fast for multi-element overlap and containment queries.
- **Write Amplification:** Inserting or updating a single row requires updating the posting lists for _every single token/element_ in that row. GIN writes are significantly heavier than B-Tree writes. (Engines use a temporary `fastupdate` pending-list buffer to mitigate this).

## Storage nuances

**Generally, ### GIN indexes expected to use much more space than [[B-Tree Index]]es.**

Because of element multiplication, If you insert 1,000,000 rows into a table:
- A B-Tree on `user_id` contains **1,000,000 index entries**.
- A GIN index on a `tags` column where each row has 10 tags contains **10,000,000 index references**.

However, GIN uses internal compression: When a specific key (like `'popular_tag'`) appears in hundreds of thousands of rows, GIN doesn't store the word `'popular_tag'` repeatedly. It stores the word once, followed by a heavily compressed **[[#Posting List]]** (or a B-tree-like structure called a **[[#Posting Tree]]**) containing the compressed integer IDs of all those rows.

**In short:** posting list = small inline array of matching row locations; posting tree = that same concept scaled up into its own B-tree once the list gets too big to store inline.

## Posting List

A simple, sorted array of TIDs (row pointers) stored inline, right next to the key, in the index's B-tree entry. Compact and fast for keys that appear in relatively few rows. Compressed using variable-byte / delta encoding since TIDs are stored as differences from the previous one.

## Posting Tree

When a posting list grows too large to fit inline (exceeds about 1/3 of a page), GIN converts it into a **posting tree**: a separate B-tree structure of its own, made up of pages full of TIDs, with the original key's entry now just pointing to the root of this tree.

This lets very common keys (e.g., a word appearing in thousands of rows) scale without bloating the main index structure.

Supports efficient insertion, sequential scanning, and merging, similar to a regular B-tree but specialized for storing sorted TID lists.

## Relation to Bitmap

When you search for multiple values using a GIN index:

```sql
WHERE tags @> ARRAY['python', 'postgres']
```

1. The GIN index finds the posting list for `'python'`: e.g., Row IDs `[1, 5, 9, 12, 100]`.
2. It finds the posting list for `'postgres'`: e.g., Row IDs `[5, 8, 9, 50, 100]`.
3. The engine creates an in-memory **TID [[Bitmap]]** for each list. In this bitmap, each bit represents a physical page or row.
4. It performs a bitwise **`AND`** operation on the two bitmaps:   Result: Rows `[5, 9, 100]`
5. In PostgreSQL execution plans, this is shown as: 
	- **`Bitmap Index Scan`** (scans the GIN structure to build the bitmap).
    - Followed by **`Bitmap Heap Scan`** (reads the matched physical pages from the table heap in physical disk order to minimize disk arm movement/random I/O).

## Real-World Scenario 1: E-commerce Product Tags

You have a table `products`:

|id|name|tags (`text[]`)|
|---|---|---|
|1|iPhone 13|`['apple', 'phone', 'sale']`|
|2|Galaxy S21|`['samsung', 'phone']`|
|3|iPad Air|`['apple', 'tablet', 'sale']`|

You want to run:


```sql
SELECT * FROM products WHERE tags @> ARRAY['sale']; -- "@>" means "contains"
```

- **With B-Tree:** Useless. A B-Tree can only compare the _entire array_ as a single giant string. It cannot see inside the array. The DB must do a Full Table Scan.
- **With GIN:** The index breaks down the arrays and stores:
    - `'apple'` → `[Row 1, Row 3]`
    - `'phone'` → `[Row 1, Row 2]`
    - `'sale'` → `[Row 1, Row 3]`
    - `'samsung'` → `[Row 2]`
    - `'tablet'` → `[Row 3]`

The query for `'sale'` immediately jumps straight to `[Row 1, Row 3]`.

#### Real-World Scenario 2: JSONB User Metadata

You store arbitrary user settings in Postgres:

```sql
-- User 1: {"theme": "dark", "notifications": true}
-- User 2: {"theme": "light", "beta_tester": true}

SELECT * FROM users WHERE metadata @> '{"beta_tester": true}';
```

A GIN index decomposes every key/value inside the JSONB document so you can query arbitrary keys at high speed without creating 50 separate columns.