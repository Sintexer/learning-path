---
flashcards:
  q-66dt: { nid: 1795356727441, hash: gkku8uah, sync: qsggwtjq }
  q-wviw: { nid: 1795356727467, hash: vh2i59z5, sync: 6ih9nkdk }
---

A relational database organizes data into **tables**. Data in one table can *link* to data in other tables to create **relationships** - hence, the relational part of the name.  
  
A table stores data in **rows** and **columns**. A row, often called a record, contains all information about a specific entry. Columns describe attributes of an entry. 

The tables, rows, columns, and relationships between them are called a logical schema. With relational databases, a schema is fixed (*schema-on-write*). After the database is operational, changing the schema requires disciplined [[Schema Migration]] practices. Because of this, most of the data modeling is done up front before the database is active.

## Query optimization 

In a relational database, the query optimizer automatically decides which parts of the query to execute in which order, and which [[Database Index|indexes]] to use. So the database automatically updates the access path as indexes change, and application code never has to be updated for this purpose (see [[Query Languages for Data]]).

### Relational database benefits

1. Complex [[SQL]] queries and ad-hoc joins.
2. Reduced redundancy — through data normalization (1NF–3NF) and foreign-key integrity.
3. Familiarity — mature ecosystem, tooling, and broad engineering expertise.
4. Accuracy — [[ACID]] [[Transaction|transactions]] and [[MVCC]] concurrency control.

## Use cases

- **Applications with well-defined, relational data** (especially many-to-many relationships).
- **Applications that need persistent storage** and strong [[ACID]] guarantees (e.g., ledgers, orders, inventory, auth).

> [!warning] Failure modes & trade-offs
> - **[[Impedance mismatch]]:** Mapping deep OOP object graphs onto normalized tables via ORMs ([[Hibernate]]/[[JPA]]) can cause N+1 queries and subtle performance issues.
> - **Over-normalization on read-heavy paths:** Joining 10+ normalized tables on a hot endpoint hurts tail latency; selective denormalization, materialized views, or `jsonb` columns in [[PostgreSQL]] often strike a better balance.

## Flashcards

Why does a [[Relational Database]] let you optimize query performance by adding [[Database Index|indexes]] without rewriting application code?::Because [[SQL]] is declarative — the query optimizer automatically picks the best access path and join strategy whenever indexes or table statistics change.
^q-66dt

What is the main advantage of normalizing data across relational tables instead of duplicating it?::It eliminates update anomalies and duplicate data, keeping many-to-one and many-to-many relationships consistent via foreign keys.
^q-wviw

## Related

- [[RDBMS]]
- [[SQL]]
- [[SQL vs NoSQL]]
- [[PostgreSQL]]
- [[ACID]]
- [[Impedance mismatch]]
- [[Schema Migration]]
- [[Database Index]]