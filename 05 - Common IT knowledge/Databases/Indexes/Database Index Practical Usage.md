
## When to use indexes

**Use [[Database Index|indexes]] when:**

- Column is frequently used in WHERE clauses
- Column is used in JOIN conditions
- Column is used for sorting (ORDER BY)
- Column is used in GROUP BY operations
- You're searching on text columns with LIKE patterns (prefix-based)
- Query performance is a bottleneck (measured with EXPLAIN ANALYZE)

**Avoid indexes when:**

- Column has low cardinality (few unique values: true/false, active/inactive)
- Writes heavily outnumber reads (INSERT/UPDATE/DELETE)
- Table is small (<10k rows—sequential scan may be faster)
- Column values are NULL frequently (some index types skip NULLs)
- You need to update that column often

## Single-column indexes

The simplest index type. Create on columns used directly in WHERE clauses.

```sql
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_created_at ON users(created_at);
```

**Query that benefits:**

```sql
SELECT * FROM users WHERE email = 'john@example.com';
SELECT * FROM users WHERE created_at > '2024-01-01';
```

## Multi-column indexes (composite indexes)

When multiple columns are searched together, a composite index is more efficient than separate single-column indexes.

### The critical rule: order matters

Create the index with columns in this priority order:

1. **Equality first** (WHERE col = value)
2. **Range conditions second** (WHERE col > value)
3. **Sort/filter third** (ORDER BY, unused in WHERE)

### Example: Index on name + surname

**Wrong order:**

```sql
-- DON'T do this
CREATE INDEX idx_name_surname ON users(surname, name);
```

**Correct order:**

```sql
-- DO this
CREATE INDEX idx_name_surname ON users(name, surname);
```

**Why?** When you search for a person, you typically query:

```sql
-- Most common: exact name + exact surname
SELECT * FROM users WHERE name = 'John' AND surname = 'Doe';

-- Also common: just name, then filter by surname
SELECT * FROM users WHERE name = 'John' AND surname LIKE 'D%';

-- Less common: surname without name
SELECT * FROM users WHERE surname = 'Doe';
```

In the correct index (name, surname):

- A query filtering by name uses the first column efficiently
- A query filtering by name + surname uses both columns
- The index can also sort by surname _after_ filtering by name

In the wrong index (surname, name):

- A query filtering by surname uses the first column
- A query filtering by name alone **cannot use this index** (can't skip the first column)
- This forces a table scan when searching by name only

### How composite indexes work: the tree principle

Think of a composite index as a tree structure:

```
Index (name, surname):
├─ "Alice"
│  ├─ "Anderson"
│  ├─ "Arnold"
│  └─ "Austin"
├─ "Bob"
│  ├─ "Baker"
│  ├─ "Brown"
│  └─ "Bruno"
└─ "Charlie"
   ├─ "Chapman"
   └─ "Chen"
```

When you search for `name = 'Alice'`, the index jumps to that branch and scans all surnames under it. When you search for `name = 'Bob' AND surname = 'Brown'`, it navigates to Bob → Brown directly.

But if you search for `surname = 'Brown'` alone, the index can't skip the "name" level—it would need to scan every branch, defeating the index's purpose.

### Practical composite index scenarios

**Scenario 1: Search by first name, then surname**

```sql
-- Query
SELECT * FROM users WHERE name = 'John' AND surname = 'Doe';

-- Index
CREATE INDEX idx_users_search ON users(name, surname);
```

**Scenario 2: Search by name, filter by created_at (range)**

```sql
-- Query
SELECT * FROM users 
WHERE name = 'John' AND created_at > '2024-01-01'
ORDER BY created_at;

-- Index: equality first, range second
CREATE INDEX idx_users_name_date ON users(name, created_at);
```

**Scenario 3: Search by name + status, sorted by created_at**

```sql
-- Query
SELECT * FROM users 
WHERE name = 'John' AND status = 'active'
ORDER BY created_at DESC;

-- Index: both equalities, then sort column
CREATE INDEX idx_users_filter_sort ON users(name, status, created_at);
```

## PostgreSQL index types

### B-tree (default)

The standard index. Works for all comparison operators: =, <, >, <=, >=, BETWEEN.

```sql
CREATE INDEX idx_email ON users(email);  -- B-tree by default
CREATE INDEX idx_email_btree ON users USING BTREE (email);  -- explicit
```

**Best for:** Exact matches, ranges, sorting.

### Hash

Only supports equality (=). Faster for exact lookups on very large indexes, but not commonly used.

```sql
CREATE INDEX idx_status_hash ON users USING HASH (status);
```

**Best for:** Exact equality only, massive tables (rarely needed).

### BRIN (Block Range Index)

Stores min/max values for ranges of rows. Extremely small and fast for sorted data.

```sql
CREATE INDEX idx_created_at_brin ON users USING BRIN (created_at);
```

**Best for:** Very large tables with naturally sorted columns (timestamps, IDs). Takes 1% the space of B-tree.

### GiST (Generalized Search Tree)

Supports geometric and range data.

```sql
CREATE INDEX idx_location_gist ON locations USING GIST (coordinates);
```

**Best for:** Geospatial queries, full-text search.

### GIN (Generalized Inverted Index)

For array and full-text search columns.

```sql
CREATE INDEX idx_tags_gin ON posts USING GIN (tags);
CREATE INDEX idx_content_gin ON posts USING GIN (to_tsvector('english', content));
```

**Best for:** Arrays, JSONB, full-text search.

## PostgreSQL index configuration

### Creating indexes

```sql
-- Simple index
CREATE INDEX idx_email ON users(email);

-- Composite index
CREATE INDEX idx_name_surname ON users(name, surname);

-- Partial index (only index rows matching a condition)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Expression index (index computed values)
CREATE INDEX idx_email_lower ON users(LOWER(email));  -- case-insensitive search

-- With descending sort
CREATE INDEX idx_created_desc ON users(created_at DESC);

-- Multi-column with mixed sort
CREATE INDEX idx_mixed ON users(name ASC, created_at DESC);

-- Unique index (enforces uniqueness)
CREATE UNIQUE INDEX idx_email_unique ON users(email);
```

### Partial indexes (very useful)

Only index rows matching a condition. Saves space and improves write performance.

```sql
-- Instead of indexing all users, only index active ones
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Query must match the WHERE condition to use the index
SELECT * FROM users WHERE status = 'active' AND email = 'john@example.com';
```

### Expression indexes

Index computed values, not raw column values.

```sql
-- Case-insensitive email search
CREATE INDEX idx_email_lower ON users(LOWER(email));

-- Query must use the same expression
SELECT * FROM users WHERE LOWER(email) = LOWER('JOHN@EXAMPLE.COM');
```

### Listing indexes

```sql
-- Show all indexes
SELECT schemaname, tablename, indexname FROM pg_indexes 
WHERE tablename = 'users';

-- Check index size
SELECT schemaname, tablename, indexname, 
  pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_indexes 
JOIN pg_class ON indexname = relname
WHERE tablename = 'users';
```

### Dropping indexes

```sql
-- Drop an index
DROP INDEX idx_email;

-- If you're not sure the index exists
DROP INDEX IF EXISTS idx_email;

-- Drop and cascade (if other objects depend on it)
DROP INDEX idx_email CASCADE;
```

---

## Monitoring and optimization

### Check if an index is being used

```sql
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'john@example.com';
```

Look for **Index Scan** or **Index Only Scan** in the output. If you see **Seq Scan**, the index isn't being used.

Example output:

```
Index Scan using idx_email on users  (cost=0.29..8.30 rows=1 width=200)
  Index Cond: (email = 'john@example.com'::text)
```

### Unused indexes

Find indexes that consume space but are never used:

```sql
SELECT schemaname, tablename, indexname
FROM pg_indexes
WHERE schemaname = 'public'
EXCEPT
SELECT schemaname, tablename, indexrelname
FROM pg_stat_user_indexes
WHERE idx_scan > 0;
```

Drop unused indexes to free space and improve write performance.

### Bloated indexes

Over time, indexes accumulate dead space. Rebuild them:

```sql
REINDEX INDEX idx_email;

-- Or rebuild all indexes on a table
REINDEX TABLE users;
```

### Statistics

PostgreSQL uses column statistics to decide whether to use an index. Update statistics after bulk inserts:

```sql
ANALYZE users;
ANALYZE users(email);  -- specific column
```

---

## Common mistakes

|Mistake|Why it's bad|Solution|
|---|---|---|
|**Indexing low-cardinality columns** (true/false, active/inactive)|Index mostly useless; sequential scan is faster|Skip the index; use partial indexes if needed|
|**Wrong column order in composite index**|Can't use index when filtering by later columns|Put equality conditions first|
|**Indexing every column**|Slows down writes, bloats table storage|Index only frequently-queried columns|
|**Not updating statistics**|PostgreSQL thinks index is useless and doesn't use it|Run ANALYZE after bulk inserts|
|**Using LIKE without prefix** (LIKE '%text')|Can't use B-tree index|Use full-text search (GIN) instead|
|**Forgetting about NULL values**|Index skips NULLs; queries with IS NULL don't use index|Use partial indexes or GIN with nulls|

---

## Real-world example: Building a user search

### Requirements

- Search by email (exact match)
- Search by name and surname (exact + partial)
- Filter by status
- Sort by created_at

### Solution

```sql
-- Email search (unique index)
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Name + surname search
CREATE INDEX idx_users_name_surname ON users(name, surname);

-- Active users filtered by email (partial index)
CREATE INDEX idx_active_users_email 
ON users(email) 
WHERE status = 'active';

-- Name + status + date (covers filtering + sorting)
CREATE INDEX idx_users_filter_sort 
ON users(name, status, created_at DESC);

-- Case-insensitive email search
CREATE INDEX idx_users_email_lower 
ON users(LOWER(email));
```

### Queries using these indexes

```sql
-- Uses idx_users_email
SELECT * FROM users WHERE email = 'john@example.com';

-- Uses idx_users_name_surname
SELECT * FROM users WHERE name = 'John' AND surname = 'Doe';

-- Uses idx_active_users_email
SELECT * FROM users WHERE status = 'active' AND email = 'john@example.com';

-- Uses idx_users_filter_sort
SELECT * FROM users 
WHERE name = 'John' AND status = 'active'
ORDER BY created_at DESC;

-- Uses idx_users_email_lower
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
```

---

## Summary checklist

- ✓ Index columns frequently used in WHERE, JOIN, ORDER BY, GROUP BY
- ✓ Skip low-cardinality columns
- ✓ Order composite indexes: equality → range → sort
- ✓ Use partial indexes for filtered data
- ✓ Use expression indexes for computed searches
- ✓ Monitor with EXPLAIN ANALYZE
- ✓ Drop unused indexes
- ✓ Rebuild bloated indexes with REINDEX
- ✓ Update statistics with ANALYZE after bulk inserts