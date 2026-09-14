An **index skip scan** is a database query optimization technique that lets the database use a multi-column (composite) index even when the query does not filter on the first column (the leading edge)

Imagine the column A only has 3 possible values in the entire table: `'CA'`, `'NY'`, and `'TX'`.

A standard scan gives up because A is missing. But a smart database engine (like MySQL 8.0, Oracle, or Postgres with specialized planning) can do an **Index Skip Scan**:

1. It pretends the query is: `WHERE A = 'CA' AND B = 10 AND C = 20` → Uses the index!
2. Then it skips directly to the next distinct value of A: `WHERE A = 'NY' AND B = 10 AND C = 20` → Uses the index!
3. Then it skips to: `WHERE A = 'TX' AND B = 10 AND C = 20` → Uses the index!

Instead of reading the entire table, it "hops" through the distinct values of the leading column.

## Example

Imagine an index on `(gender, age)`:
- `gender` has only two distinct values: `'F'` and `'M'`.
- `age` has many values (1 to 100).

Your query is:

```sql
SELECT * FROM users WHERE age = 25;
```

Notice that `gender` (the leading column) is missing from the `WHERE` clause.

## How a standard B-Tree behaves:

Because the [[B-Tree Index]] is sorted primarily by `gender`, all `'F'` rows come first, and all `'M'` rows come later. The engine cannot know where `age = 25` is without checking all branches. It gives up and scans the whole table.

The engine knows `gender` has very few distinct values (low cardinality). So, behind the scenes, the engine splits your query into two targeted sub-searches:

```SQL
Search 1: WHERE gender = 'F' AND age = 25  --> [B-Tree jump directly to result]
Search 2: WHERE gender = 'M' AND age = 25  --> [B-Tree jump directly to result]
```

It **skips** the rest of the tree. Instead of scanning 1,000,000 rows sequentially, it performs two fast logarithmic seeks down the index.

> [!important] 
> Skip scan only works when the skipped leading column has **low cardinality** (very few unique values, like status, gender, country code).