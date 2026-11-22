---
flashcards:
  q-g5u8: { nid: 1795356728718, hash: dshj7h42, sync: qajf5sz4 }
  q-em93: { nid: 1795356728742, hash: 7hdfv87n, sync: yd7nt92a }
---

Most application development today is done in object-oriented programming languages, which leads to a common criticism of the [[SQL]] / [[Relational Database|relational]] data model: if data is stored in relational tables, an awkward translation layer is required between the objects in application code and the database model of tables, rows, and columns. The disconnect between the two models is called **object-relational impedance mismatch**.

## Where the Mismatch Shows Up

- **Granularity & nesting:** Objects naturally contain nested trees and collections (e.g., `Order` → `List<OrderItem>`), whereas relational tables normalize them into separate flat tables joined by foreign keys.
- **Inheritance vs. tables:** OOP has class hierarchies (`Payment` → `CardPayment`, `BankTransfer`); SQL has no native inheritance, forcing awkward mapping strategies (single-table with nullable columns, joined-table with extra joins, or table-per-class).
- **Identity vs. equality:** In Java/OOP, identity is object reference (`==`) or `equals()`; in a database, identity is the primary key (causing subtle bugs with unsaved entities in `Set`/`HashMap`).
- **Navigation vs. set operations:** OOP traverses references (`user.getOrders().get(0).getItems()`), whereas relational databases are built for set-based queries (`JOIN`, `WHERE`).

## How Systems Mitigate It

- **ORM frameworks ([[JPA]] / [[Hibernate]]):** Automate translation between objects and rows, though leaky abstractions still require understanding the underlying SQL.
- **[[Document Database|Document databases]] ([[MongoDB]]) or `jsonb` columns ([[PostgreSQL]]):** Store tree-structured aggregates as a single JSON document, eliminating joins for self-contained one-to-many data.
- **Explicit query/DTO mappers (jOOQ, MyBatis, Spring JDBC):** Embrace SQL directly for read paths and map result sets straight into immutable DTOs (pairs well with [[CQRS]]).

> [!warning] Failure modes & trade-offs
> - **The N+1 query problem:** Navigating lazy-loaded ORM object graphs (`order.getItems()`) inside a loop fires 1 query for the parent list plus $N$ hidden round-trips to the DB.
> - **Over-fetching & accidental write amplification:** Loading an entire entity graph just to update one field (or return 3 fields in a REST API) wastes network bandwidth, memory, and [[Database Lock|locks]].

## Flashcards

What is **object-relational impedance mismatch**?::The structural friction between in-memory object-oriented models (nested graphs, inheritance, reference navigation) and [[Relational Database|relational]] tables (flat normalized rows, foreign keys, set-based queries).
^q-g5u8

What classic performance bug occurs when an ORM like [[Hibernate]] hides impedance mismatch behind lazy-loaded object navigation?::The **N+1 query problem** — fetching $N$ parent rows in 1 query and then triggering $N$ separate follow-up queries when iterating over a lazy association.
^q-em93

## Related

- [[Relational Database]]
- [[Document Database]]
- [[JPA]]
- [[Hibernate]]
- [[SQL]]
- [[CQRS]]
