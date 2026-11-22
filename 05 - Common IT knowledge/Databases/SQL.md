---
flashcards:
  q-6gpd: { nid: 1795356726792, hash: kcpr576p, sync: q8nuxf5w }
---

> AKA **Structured Query Language** or **SEQUEL**

**SQL (Structured Query Language)** is a powerful and versatile programming language designed specifically for managing data stored in [[Relational Database|relational databases]]. It allows users to perform various operations like creating and modifying [[Database|database]] structures, inserting and updating data, retrieving specific information based on complex criteria, and analyzing data for insights. SQL's standardized syntax and wide adoption across different database systems make it an essential skill for data analysts, developers, and anyone working with large amounts of structured data.

> [!example]
> Select query:
> 
> ```sql
> SELECT * FROM table_name
> ```

However, the power of SQL queries is in creating more complex queries that pull data from several tables to identify patterns and answers to business problems. For example, querying the sales table and the books table together to see sales in relation to an author’s books. Querying tables together to better understand their relationships is made possible by a "join".

## SQL Sub-Languages

- **DDL (Data Definition Language):** `CREATE`, `ALTER`, `DROP` — defines and evolves schemas (see [[Schema Migration]]).
- **DML (Data Manipulation Language):** `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` — reads and mutates rows.
- **TCL (Transaction Control Language):** `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` — controls [[Transaction]] boundaries.
- **DCL (Data Control Language):** `GRANT`, `REVOKE` — manages access permissions.

> [!tip] Production reality
> Mastering production SQL goes beyond syntax: understand [[Logical Query Processing Order]], [[Anti-join and NOT IN trap|`NULL` three-valued logic]], [[Grouping]] vs. [[Window Functions]], and how to read `EXPLAIN (ANALYZE, BUFFERS)` to distinguish [[IO-Bound vs CPU-Bound]] bottlenecks.

## Flashcards

What are the four standard sub-languages of [[SQL]]?::DDL (`CREATE`/`ALTER`/`DROP`), DML (`SELECT`/`INSERT`/`UPDATE`/`DELETE`), TCL (`BEGIN`/`COMMIT`/`ROLLBACK`), and DCL (`GRANT`/`REVOKE`).
^q-6gpd

## Related

- [[Query Languages for Data]]
- [[Logical Query Processing Order]]
- [[Aggregate Functions]]
- [[Grouping]]
- [[Window Functions]]
- [[Anti-join and NOT IN trap]]
- [[IO-Bound vs CPU-Bound]]
- [[Relational Database]]
- [[RDBMS]]