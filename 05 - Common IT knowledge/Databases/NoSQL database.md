**Not Only SQL** — an umbrella term for non-relational [[Database|databases]] designed for flexible schemas, specialized data models, high write throughput, or massive [[Horizontal scaling]] where traditional single-node [[RDBMS]] constraints become bottlenecks.

## The Four Major Families

| Family | Data Model | Best Fit | Examples |
| :--- | :--- | :--- | :--- |
| **Key-Value** | Opaque `key -> value` | Caching, sessions, feature flags, simple lookups | [[Redis]], [[Memcached]], DynamoDB |
| **[[Document Database\|Document]]** | Nested JSON/BSON documents | Self-contained aggregates, content/catalogs, heterogeneous schemas | [[MongoDB]], Couchbase |
| **Wide-Column** | Partition key + sorted clustering columns | High-write time-series, event logs, multi-region availability | [[Apache Cassandra]], ScyllaDB, HBase |
| **[[Graph database\|Graph]]** | Vertices & labeled edges | Social networks, fraud rings, recommendations, knowledge graphs | Neo4j, Amazon Neptune |

## Core Architectural Characteristics

- **Query-driven modeling & denormalization:** Rather than normalizing to 3NF and joining at read time, NoSQL stores pre-shape data for known access patterns.
- **Native [[Data partitioning|sharding]] & [[Replication]]:** Built from day one to distribute partitions across commodity nodes (often via [[Consistent Hashing]] and [[Leaderless Replication]] or shard-per-replica-set).
- **Tunable consistency:** Many NoSQL stores trade strict [[Linearizability]] / multi-row [[ACID]] for lower latency and higher [[Availability]] under partitions (see [[CAP Theorem]], [[Consistency Levels]], [[Eventual Consistency]]).

> [!warning] Failure modes & trade-offs
> - **Ad-hoc query pain:** Because tables are modeled around fixed access patterns, a new product requirement ("filter orders by promo code and sort by region") can require expensive full-cluster scans or backfilling a brand-new denormalized table.
> - **Application-side join & integrity burden:** Without foreign keys or cross-record transactions, keeping denormalized copies in sync falls on application code (and is vulnerable to [[Dual-Write]] bugs).
> - **Don't pick NoSQL "for scale" prematurely:** A well-indexed [[PostgreSQL]] instance handles tens of thousands of QPS and multi-terabyte datasets with far simpler operational and transactional guarantees (see [[SQL vs NoSQL]]).

## Flashcards

What are the four main categories of [[NoSQL database|NoSQL databases]]?::Key-Value ([[Redis]]), [[Document Database|Document]] ([[MongoDB]]), Wide-Column ([[Apache Cassandra]]), and [[Graph database|Graph]] (Neo4j).

What is the biggest long-term trade-off of denormalized, query-driven NoSQL data modeling?::New ad-hoc access patterns are hard to serve without backfilling new tables/indexes, and keeping duplicated data consistent across entities shifts complexity into application code.

## Related

- [[SQL vs NoSQL]]
- [[Document Database]]
- [[Graph database]]
- [[Apache Cassandra]]
- [[MongoDB]]
- [[Redis]]
- [[CAP Theorem]]
- [[Consistency Levels]]
- [[Data partitioning]]
