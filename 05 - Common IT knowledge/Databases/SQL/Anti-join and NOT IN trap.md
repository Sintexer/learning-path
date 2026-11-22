An **Anti-Join** returns rows from the left table that have **no matching row** in the right table.  
_Example: "Find all users who have NEVER placed an order."_

There are 3 standard ways to write this in SQL, and **one of them has a dangerous trap**:

```sql
-- Method 1: NOT EXISTS (Recommended)
SELECT * FROM users u 
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- Method 2: LEFT JOIN ... WHERE IS NULL (Classical Anti-Join)
SELECT u.* FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
WHERE o.id IS NULL;

-- Method 3: NOT IN (THE DANGER ZONE)
SELECT * FROM users 
WHERE id NOT IN (SELECT user_id FROM orders);
```

## Why is `NOT IN` dangerous?

If the subquery inside `NOT IN` returns **even a single `NULL` value**, the entire query returns **ZERO rows (empty result)**!

**The Boolean Logic Under the Hood:**

- In SQL, comparisons with `NULL` evaluate to `UNKNOWN` (Three-Valued Logic).
- `WHERE id NOT IN (1, 2, NULL)` is converted to:  
    `WHERE (id != 1) AND (id != 2) AND (id != NULL)`
- Because `id != NULL` evaluates to `UNKNOWN`, the whole `AND` condition evaluates to `UNKNOWN` or `FALSE`.
- Not a single row is returned, leading to a silent, catastrophic production bug.

> [!tip] Rule
> Always use **`NOT EXISTS`** or **`LEFT JOIN ... WHERE IS NULL`**. The optimizer converts both of these into a specialized internal **Hash Anti-Join** execution node, which is safe, fast, and ignores `NULL` traps.

> [!warning] Failure modes & trade-offs
> - **Nullable columns sneak in later:** Even if `orders.user_id` is non-null today, a future [[Schema Migration]] or left-joined CTE that introduces a single `NULL` will silently turn a working `NOT IN` query into an empty-result bug.
> - **`LEFT JOIN` trap with `IS NULL`:** Make sure the `IS NULL` check targets a `NOT NULL` column on the right table (such as its primary key `o.id`), not a nullable attribute that could legitimately be `NULL` on matched rows.

## Flashcards

Why does `WHERE id NOT IN (SELECT user_id FROM orders)` return **zero rows** if the subquery contains even one `NULL`?::SQL uses three-valued logic: `id NOT IN (1, NULL)` expands to `id != 1 AND id != NULL`; since `id != NULL` evaluates to `UNKNOWN`, the whole `AND` never evaluates to `TRUE`.

Prefer ==`NOT EXISTS`== or ==`LEFT JOIN ... WHERE right_pk IS NULL`== over `NOT IN` for SQL anti-joins to stay immune to `NULL` values.

## Related

- [[SQL]]
- [[Logical Query Processing Order]]
- [[Database Index Practical Usage]]
- [[Schema Migration]]