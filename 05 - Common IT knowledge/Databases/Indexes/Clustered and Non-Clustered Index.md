---
flashcards:
  q-ajcb: { nid: 1795356728292, hash: puh8epdj, sync: hgrwnh7v }
  q-bmqz: { nid: 1795356728319, hash: 3mj2m7jv, sync: rsgt4h9j }
  q-j27j: { nid: 1795356728341, hash: qj3yj6va, sync: 232h8pk5 }
---

## Clustered Index

The table data itself is **physically stored in the order of the index**, and the index's leaf nodes contain the **actual row data** — not a pointer to it. There's no separate [[DB Glossary#Table Heap|Heap]] at all; the index _is_ the table storage.

- A table can have **at most one** clustered index (data can only be physically sorted one way).
- In **SQL Server**, this is literally called a "clustered index."
- In **MySQL/InnoDB**, the **primary key** is always the clustered index (called the "clustered key"); if no primary key is defined, InnoDB picks/generates one internally.
- In **Oracle**, this concept is called an **Index-Organized Table (IOT)** — the table's rows live inside the B-tree structure of the index.
- **Benefit:** lookups by the clustered key are extremely fast since the data is found in one step, and range scans on that key are sequential on disk.

## Non-Clustered / Secondary Index

The index is a **separate structure** from the table's storage (the "heap"). Each leaf entry stores the indexed column value plus a **pointer/reference** to where the actual row lives:

- In [[PostgreSQL]], all indexes work this way — PostgreSQL has no clustered index concept by default (it does have a one-time `CLUSTER` command to physically reorder the heap to match an index, but it isn't maintained automatically afterward).
- In **SQL Server/MySQL**, these are called **non-clustered** or **secondary indexes**. The pointer is either:
    - A physical **row ID / RID** (heap tables), or
    - The **clustered key value** itself (if the table has a clustered index) — meaning a secondary index lookup requires an extra step ("bookmark lookup") to fetch the full row via the clustered index.
- A table can have **many** non-clustered indexes.
- **Trade-off:** an extra hop (index → heap) is needed to fetch columns not covered by the index — this is exactly the "heap fetch" that a **covering index** (discussed earlier) avoids.

## Quick Summary

| |Clustered Index|Non-Clustered / Secondary Index|
|---|---|---|
|Storage|Row data lives in the index leaves|Index stores a pointer/ref to the row (heap or clustered key)|
|Count per table|One|Many|
|Terminology|Clustered index (SQL Server), Index-Organized Table (Oracle), primary key = clustered key (InnoDB)|Non-clustered / secondary index; heap table (PostgreSQL default)|
|Lookup cost|Direct — data found in the index itself|Extra hop to fetch the actual row|

> [!warning] Failure modes & trade-offs
> - **InnoDB secondary index bloat from wide primary keys:** Because every secondary index in MySQL InnoDB stores the **clustered primary key** as its row pointer, using a 36-byte `CHAR(36)` UUID primary key bloats *every* secondary index on that table and doubles lookup work (secondary tree → primary tree). Prefer compact 8-byte `BIGINT` or binary `UUIDv7` keys.
> - **PostgreSQL `CLUSTER` is a one-shot lock:** Running `CLUSTER table USING idx` in [[PostgreSQL]] takes an `ACCESS EXCLUSIVE` lock, rewrites the heap once, and does *not* keep future inserts sorted.

## Flashcards

Why can a table have at most **one** clustered index?::Because a clustered index determines the physical storage order of the table's rows on disk, and rows can only be physically ordered one way.
^q-ajcb

In MySQL InnoDB, secondary index leaf nodes store ==the primary (clustered) key== as the row pointer, which means wide primary keys bloat ==every secondary index== on the table.
^q-bmqz

How does [[PostgreSQL]] store table rows compared to MySQL InnoDB?::PostgreSQL stores all tables as unordered heap files with non-clustered indexes pointing to physical tuple IDs (TIDs), whereas InnoDB stores every table clustered by its primary key.
^q-j27j

## Related

- [[Database Index]]
- [[B-Tree Index]]
- [[PostgreSQL]]
- [[DB Glossary]]