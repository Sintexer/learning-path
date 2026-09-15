
Developers write queries starting with `SELECT`. But the database engine executes the query in a completely different, strict logical order:

$$\text{FROM} \longrightarrow \text{JOIN} \longrightarrow \text{WHERE} \longrightarrow \text{GROUP BY} \longrightarrow \text{HAVING} \longrightarrow \text{SELECT} \longrightarrow \text{DISTINCT} \longrightarrow \text{ORDER BY} \longrightarrow \text{LIMIT}$$

### Why this explains common interview traps:
1. **"Why can't I use a column alias defined in `SELECT` inside the `WHERE` or `GROUP BY` clause?"**
   * *Answer:* Because `WHERE` (Step 3) and `GROUP BY` (Step 4) are evaluated **long before** the `SELECT` clause (Step 6) even exists in the engine pipeline!
2. **"Why can't I use an aggregate function like `COUNT()` in the `WHERE` clause?"**
   * *Answer:* The `WHERE` clause filters individual rows before groups are formed. Aggregates don't exist until `GROUP BY` runs.
3. **"Can I use a column alias defined in `SELECT` inside `ORDER BY`?"**
   * *Answer:* **Yes**, because `ORDER BY` (Step 8) executes **after** `SELECT` (Step 6).