---
flashcards:
  q-3tpc: { nid: 1795356727091, hash: es7fs35k, sync: bstmzznw }
  q-xiin: { nid: 1795356727118, hash: 9936kjek, sync: t8qxivyk }
---
`COUNT`, `SUM`, `AVG`, `MIN`, `MAX` — these **collapse multiple rows into a single row** per group.

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

**Key property:** you lose row-level detail. If Engineering has 50 employees, this query gives you **1 row** for Engineering with the total — the individual employee rows are gone from the output. (To keep every row while computing group totals, use [[Window Functions]] instead.)

> [!warning] Failure modes & trade-offs
> - **`NULL` handling traps:** `SUM`, `AVG`, `MIN`, `MAX`, and `COUNT(col)` all ignore `NULL`s, whereas `COUNT(*)` counts every row. `AVG(score)` over `(10, NULL)` returns `10`, not `5` — wrap with `COALESCE(score, 0)` if `NULL` means zero.
> - **Empty group result:** On 0 matching rows, `COUNT(*)` returns `0`, while `SUM`/`AVG`/`MIN`/`MAX` return `NULL`.
> - **Filter before aggregating:** Put row-level filters in `WHERE`, not `HAVING` (see [[Grouping]] and [[Logical Query Processing Order]]).

## Flashcards

How do SQL [[Aggregate Functions]] like `AVG(col)` and `COUNT(col)` treat `NULL` values compared to `COUNT(*)`?::`AVG(col)` and `COUNT(col)` ignore `NULL` values (excluding them from the denominator/count), whereas `COUNT(*)` counts all rows including `NULL`s.
^q-3tpc

What is the fundamental difference between [[Aggregate Functions]] with `GROUP BY` and [[Window Functions]] with `OVER()`?::Aggregate functions collapse multiple rows into one output row per group; window functions preserve every input row and annotate each with the computed value.
^q-xiin

## Related

- [[Grouping]]
- [[Window Functions]]
- [[Logical Query Processing Order]]
- [[SQL]]
