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