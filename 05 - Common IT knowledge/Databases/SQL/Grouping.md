This highly depends on [[Logical Query Processing Order]].
## Engine Mechanics: How `GROUP BY` Executes Physically

When you write `GROUP BY customer_id`, how does the storage engine physically build the groups? In [[PostgreSQL]] (and most relational engines), the optimizer chooses one of two strategies based on data size and indexes:

```
                          [ Incoming Tuples ]
                                   |
         -----------------------------------------------------
         |                                                   |
   [ Unsorted Data ]                                   [ Sorted Data ]
         |                                                   |
         v                                                   v
+-----------------------+                           +-----------------------+
|     HashAggregate     |                           |    GroupAggregate     |
|   (In-Memory Hash)    |                           |     (Stream-based)    |
+-----------------------+                           +-----------------------+
```

### Strategy A: `HashAggregate`
* **How it works:** The engine allocates an in-memory hash table in RAM (`work_mem`).
* For each row read from disk, it computes the hash of the grouping key (e.g., `hash(customer_id)`).
* If the key exists in the hash table, it updates the running aggregate in-place (e.g., adds to `sum` or increments `count`).
* If not, it inserts a new bucket.
* **The Danger (Disk Spill):** If the number of distinct groups is massive and exceeds `work_mem`, the hash table must spill to disk into temporary files, degrading performance.

### Strategy B: `GroupAggregate` (Sorted Stream)
* **How it works:** This strategy requires the incoming rows to be **pre-sorted** by the group key.
* The engine reads rows sequentially. Because all identical keys are adjacent, it only keeps **one active group in memory at a time**:
  1. Read Row 1 (`ID: 10`) $\to$ initialize accumulator.
  2. Read Row 2 (`ID: 10`) $\to$ add to accumulator.
  3. Read Row 3 (`ID: 11`) $\to$ Emit result for `10` immediately! Clear memory, start accumulating `11`.
* **Memory footprint:** Constant $O(1)$ memory!
* **Optimization Secret:** If you already have a **B-Tree index** on the `GROUP BY` column, the engine can use `GroupAggregate` directly by scanning the index leaf nodes, requiring zero sorting and almost zero RAM.

## `WHERE` vs. `HAVING` Under the Hood

| Feature | `WHERE` | `HAVING` |
| :--- | :--- | :--- |
| **Execution Point** | **Before** grouping and aggregation | **After** grouping and aggregation |
| **Operates On** | Individual row tuples | Aggregated group buckets |
| **Can filter on** | Table columns | Aggregated calculations (`COUNT`, `SUM`, `AVG`) |
| **Index usage** | Can leverage B-Tree/BRIN indexes to skip rows entirely | **Cannot use indexes** directly; operates on the in-memory buckets |

### The Production Performance Trap:
Look at these two queries that return the exact same answer:

```sql
-- Query A (Bad):
SELECT department_id, COUNT(*) 
FROM employees 
GROUP BY department_id 
HAVING department_id IN (1, 2);

-- Query B (Good):
SELECT department_id, COUNT(*) 
FROM employees 
WHERE department_id IN (1, 2) 
GROUP BY department_id;
```
* In **Query A**, the engine reads all 1,000,000 employees, creates groups for every single department in memory, and then discards 98% of the groups via `HAVING`.
* In **Query B**, the engine uses an index on `department_id` in the `WHERE` clause, reads only the 20,000 relevant rows, and groups only those. Query B is orders of magnitude faster.
* **Rule:** If a filter does not contain an aggregate function, put it in `WHERE`.

##  Modern SQL Aggregation Techniques

### A. `COUNT(*)` vs `COUNT(column)` vs `COUNT(1)`
* `COUNT(*)`: Counts **all rows**, regardless of whether columns contain `NULL`.
* `COUNT(1)`: Identical to `COUNT(*)`. The query optimizer converts `COUNT(1)` into `COUNT(*)` internally. There is **zero** performance difference in modern engines.
* `COUNT(column)`: Counts only rows where that specific column is **`NOT NULL`**. (Requires the engine to inspect the column value for nullability on every row).

### B. Conditional Aggregations: `FILTER (WHERE ...)` vs `CASE`
Suppose you need to count total orders, active orders, and cancelled orders in a single query.

* **The Old SQL Approach (Verbose):**
  ```sql
  SELECT 
      user_id,
      COUNT(*) AS total_orders,
      SUM(CASE WHEN status = 'ACTIVE' THEN 1 ELSE 0 END) AS active_orders,
      SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled_orders
  FROM orders
  GROUP BY user_id;
  ```
* **The Modern ANSI SQL Standard Approach (`FILTER` clause):**
  ```sql
  SELECT 
      user_id,
      COUNT(*) AS total_orders,
      COUNT(*) FILTER (WHERE status = 'ACTIVE') AS active_orders,
      COUNT(*) FILTER (WHERE status = 'CANCELLED') AS cancelled_orders
  FROM orders
  GROUP BY user_id;
  ```
  *(PostgreSQL, SQLite, and DuckDB support `FILTER` natively. MySQL still requires `SUM(CASE ...)`).*