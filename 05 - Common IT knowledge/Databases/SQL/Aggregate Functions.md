
`COUNT`, `SUM`, `AVG`, `MIN`, `MAX` — these **collapse multiple rows into a single row** per group.

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

**Key property:** you lose row-level detail. If Engineering has 50 employees, this query gives you **1 row** for Engineering with the total — the individual employee rows are gone from the output.
