> AKA **Online Transaction Processing**

An application typically looks up a small number of records by some key, using a [[Database Index|index]]. Records are inserted or updated based on the user's input. Because these applications are interactive, the access pattern became known as **online [[Transaction|transaction]] processing** (OLTP).

## Core Characteristics

- **Access pattern:** High volume of short, concurrent, random point reads and writes (milliseconds latency).
- **Storage layout:** Row-oriented [[Database Storage Engines|storage engines]] ([[B-Tree Index|B-Trees]] or [[LSM Tree|LSM-Trees]]) so reading or updating a single entity touches one contiguous record.
- **Schema:** Normalized (typically 3NF) in [[RDBMS|relational databases]] to minimize write anomalies and keep updates small.
- **Correctness focus:** Relies on [[ACID]] guarantees, [[MVCC]], and fine-grained [[Database Lock|row-level locking]] to handle concurrent user requests safely.
- **Examples:** [[PostgreSQL]], MySQL, Oracle, SQL Server, [[MongoDB]], DynamoDB.

> [!warning] Failure modes & trade-offs
> - **Unindexed queries turn OLTP into full scans:** A single missing index on a hot endpoint turns a 1 ms point lookup into a multi-second sequential scan that saturates I/O and starves other transactions.
> - **Connection exhaustion:** OLTP databases handle hundreds of active connections poorly without a connection pooler (e.g., PgBouncer, HikariCP) because each connection consumes RAM and context-switching overhead.
> - **Write amplification from over-indexing:** Every secondary index speeds up reads but slows down every `INSERT`/`UPDATE`/`DELETE` — keep OLTP indexes lean.

## Flashcards

Why do OLTP databases use row-oriented storage rather than [[Column-Oriented Storage]]?::OLTP queries fetch and update entire records by primary key, so storing all columns of a row together minimizes disk seeks.

What happens to an OLTP database when a high-QPS endpoint lacks a proper [[Database Index]]?::Point lookups degrade into full table scans, exhausting disk I/O and causing tail-latency spikes across all concurrent transactions.

## Related

- [[OLAP]]
- [[OLTP and OLAP]]
- [[Transaction]]
- [[ACID]]
- [[Database Index]]
- [[RDBMS]]
- [[PostgreSQL]]
