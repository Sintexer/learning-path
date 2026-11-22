---
flashcards:
  q-7jwt: { nid: 1795356727491, hash: j3y5iegw, sync: yugp8ijv }
  q-8k63: { nid: 1795356727519, hash: 3fgytan6, sync: m5drntu5 }
---

When the [[Relational Database|relational model]] was introduced, it included a new way of querying data:
[[SQL]] is a *declarative* query language, whereas IMS and CODASYL queried the database using *imperative* code. What does that mean?

Many commonly used programming languages are imperative (your code specifies the exact way of iterating and extracting the data).

In a declarative query language, like [[SQL]] or relational algebra, you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal. It is up to the database system’s query optimizer to decide which [[Database Index|indexes]] and which join methods to use, and in which order to execute various parts of the query.

The SQL output doesn’t guarantee any particular ordering, and so it doesn’t mind if the order changes. But if the query is written as imperative code, the database can never be sure whether the code is relying on the ordering or not. The fact that SQL is more limited in functionality gives the database much more room for automatic optimizations (including parallel execution across cores or distributed nodes).

[[Graph database|Graph databases]] use 3 other well-known query languages:
- [[Graph database#The Cypher Query Language]]
- [[Graph database#SPARQL query language]]
- [[Graph database#Datalog]]

> [!tip] Production reality
> Declarative languages decouple **what** you want from **how** it is fetched: you can add a new [[B-Tree Index]] or upgrade the [[DBMS]] optimizer and existing queries get faster without changing a single line of application code. The trade-off is occasional **plan instability** — when table statistics drift, the optimizer can suddenly switch from an Index Scan to a Seq Scan (diagnose with `EXPLAIN ANALYZE` and fix with `ANALYZE`).

## Flashcards

Why do declarative query languages like [[SQL]] enable far better database optimization than imperative data-access code?::Because they declare *what* result is wanted without constraining *how* to execute it — freeing the query optimizer to choose indexes, join algorithms, and parallel execution plans automatically.
^q-7jwt

Why is a SQL query without an explicit `ORDER BY` allowed to return rows in a different order after an upgrade or index change?::SQL treats tables as unordered sets; leaving ordering unspecified lets the optimizer pick faster physical access paths (like hash joins or parallel scans).
^q-8k63

## Related

- [[SQL]]
- [[Logical Query Processing Order]]
- [[MapReduce querying]]
- [[Graph database]]
- [[Relational Database]]