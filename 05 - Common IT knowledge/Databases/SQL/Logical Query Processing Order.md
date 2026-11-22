
Developers write queries starting with `SELECT`. But the database engine executes the query in a completely different, strict logical order:

$$\text{FROM} \longrightarrow \text{JOIN} \longrightarrow \text{WHERE} \longrightarrow \text{GROUP BY} \longrightarrow \text{HAVING} \longrightarrow \text{SELECT} \longrightarrow \text{DISTINCT} \longrightarrow \text{ORDER BY} \longrightarrow \text{LIMIT}$$

### Why this explains common interview traps:
1. **"Why can't I use a column alias defined in `SELECT` inside the `WHERE` or `GROUP BY` clause?"**
   * *Answer:* Because `WHERE` (Step 3) and `GROUP BY` (Step 4) are evaluated **long before** the `SELECT` clause (Step 6) even exists in the engine pipeline!
2. **"Why can't I use an aggregate function like `COUNT()` in the `WHERE` clause?"**
   * *Answer:* The `WHERE` clause filters individual rows before groups are formed. Aggregates don't exist until `GROUP BY` runs.
3. **"Can I use a column alias defined in `SELECT` inside `ORDER BY`?"**
   * *Answer:* **Yes**, because `ORDER BY` (Step 8) executes **after** `SELECT` (Step 6).
4. **"Why can't I filter on a [[Window Functions|window function]] in `WHERE`?"**
   * *Answer:* Window functions are evaluated during `SELECT` (Step 6), after `WHERE`, `GROUP BY`, and `HAVING`. Wrap the query in a CTE/subquery (or use `QUALIFY` in dialects like DuckDB/BigQuery/Snowflake).

> [!tip] Production reality
> Logical processing order defines **semantic validity** (what aliases/aggregates are in scope), not the physical execution plan. The query optimizer is free to push predicates down into `JOIN`s or use a [[B-Tree Index]] to satisfy `ORDER BY ... LIMIT` without sorting the entire table, as long as the logical result is identical.

## Flashcards

What is the SQL logical query processing order from first clause to last?::`FROM` → `JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`.

Why can a `SELECT` column alias be referenced in `ORDER BY` but not in `WHERE`?::Because `WHERE` runs before `SELECT` in logical query processing order, whereas `ORDER BY` runs after `SELECT`.

## Related

- [[SQL]]
- [[Grouping]]
- [[Aggregate Functions]]
- [[Window Functions]]
- [[Anti-join and NOT IN trap]]