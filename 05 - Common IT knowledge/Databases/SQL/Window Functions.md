
**Definition:** A window function performs a calculation **across a set of related rows** (called a "window"), but **without collapsing those rows** — every original row stays in the output, each annotated with the computed value.

```sql
SELECT 
    employee_id,
    department,
    salary,
    SUM(salary) OVER (PARTITION BY department) AS dept_total
FROM employees;
```

## `OVER()` clause

This is what makes something a window function — the `OVER()` clause:

```sql
function_name(...) OVER (
    PARTITION BY column   -- optional: defines the "grouping" for the window, without collapsing rows
    ORDER BY column        -- optional: defines row order within each partition (needed for ranking/lag/lead)
    [frame clause]          -- optional: ROWS/RANGE — how many rows around current row are included
)
```

- **`PARTITION BY`** — like `GROUP BY`, but doesn't collapse rows. It just defines which rows belong "together" for the calculation (e.g., calculate per-department totals, but keep every employee row).
- **`ORDER BY`** (inside `OVER`) — defines the sequence rows are processed in *within* each partition. Required for anything sequential — ranking, running totals, previous/next row.

### 1. Ranking functions

```sql
SELECT employee_id, salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK()       OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```
- **`ROW_NUMBER()`** — unique sequential number, always distinct even for ties (1,2,3,4...)
- **`RANK()`** — same rank for ties, but **skips** the next number(s) (1,2,2,4...)
- **`DENSE_RANK()`** — same rank for ties, but **doesn't skip** (1,2,2,3...)

This tie-handling difference is a classic interview question by itself — worth memorizing that exact example.

### 2. Offset functions — `LAG` / `LEAD`
```sql
SELECT employee_id, salary,
    LAG(salary, 1)  OVER (ORDER BY hire_date) AS prev_employee_salary,
    LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_employee_salary
FROM employees;
```
- **`LAG`** — look at a **previous** row's value
- **`LEAD`** — look at a **following** row's value

**Real use case to have ready:** "Comparing each day's revenue to the previous day's revenue for a day-over-day change calculation" — this is the single most common real-world use of `LAG`.

### 3. Aggregate functions used as window functions (this is where your SUM/COUNT knowledge connects directly)
```sql
SELECT order_id, order_date, amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```
This is the **running total** pattern — same `SUM` you already know, just used with `OVER()` instead of `GROUP BY`, so it accumulates row-by-row instead of collapsing.

### 4. Distribution functions (nice-to-have)
- **`NTILE(n)`** — splits rows into `n` roughly equal buckets (e.g., quartiles)
- **`FIRST_VALUE` / `LAST_VALUE`** — get the first/last value in the window frame

---

## The frame clause (nice-to-have depth, but good to mention if pushed)

```sql
SUM(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) AS rolling_3_day_sum
```
This computes a **moving/rolling window** — e.g., a 3-day rolling sum, not a full running total. If asked "how would you compute a 7-day moving average," this frame clause is the mechanism.

---

## The trap question interviewers love: "Can you use a window function in a `WHERE` clause?"

**Answer: No — and knowing why shows real understanding of SQL execution order.**

SQL logical execution order is roughly:
```
FROM → WHERE → GROUP BY → HAVING → SELECT (window functions computed here) → ORDER BY
```

Window functions are computed **after** `WHERE`/`GROUP BY`/`HAVING`, as part of the `SELECT` list — so you **cannot** filter on a window function's result in the same query's `WHERE` clause (it doesn't exist yet at that point in execution). You have to wrap it:

```sql
SELECT * FROM (
    SELECT employee_id, salary,
        RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 10;
```
Or use a CTE (`WITH` clause) instead of a subquery — same idea.

**This trap connects directly to your mystery term** — this execution-order concept is exactly the kind of thing that comes up right around aggregate/window function discussions. Does "query execution order" or needing a **subquery/CTE wrapper to filter on a window function** ring any bells as the term you're blanking on? Also throwing out a few other likely candidates from that same conversational neighborhood: **`HAVING`** (filtering after `GROUP BY`), **`ROLLUP`/`CUBE`** (multi-level aggregation), **correlated subquery**, or **CTE** (Common Table Expression, the `WITH` keyword). Any of these sound like it?

---

## The one-liner to nail this distinction in an interview

> "Aggregate functions with `GROUP BY` collapse multiple rows into one row per group. Window functions compute a value across a related set of rows using `OVER()`, but keep every original row in the output — that's the fundamental difference. You use `PARTITION BY` instead of `GROUP BY` when you want the grouping logic without losing row-level detail."

---

## Quick check

1. Given an `orders` table with `customer_id`, `order_date`, `amount` — write (verbally or in text) the query to get, for each order, the **running total** of that customer's spending up to that order, ordered by date.
2. Did any of the terms I listed (HAVING, ROLLUP/CUBE, correlated subquery, CTE, execution order) spark recognition as your missing term?