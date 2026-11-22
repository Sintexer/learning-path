---
flashcards:
  q-qduc: { nid: 1795356726120, hash: 9p42gdf5, sync: 6ub2z3ya }
  q-262e: { nid: 1795356726148, hash: ykevsmxt, sync: gwtwywz6 }
---

- [[OLTP]]
- [[OLAP]]

| Property               | OLTP (Online Transaction Processing)                       | OLAP (Online Analytical Processing)                                    |
| :--------------------- | :--------------------------------------------------------- | :--------------------------------------------------------------------- |
| **Primary Use Case**   | Day-to-day operations, web apps, user interactions         | Business intelligence, reporting, data science                         |
| **Read Pattern**       | Small number of records per query, fetched by key          | Scans millions of rows over a few specific columns                     |
| **Write Pattern**      | Frequent, random writes and updates from end-users         | Bulk loads ([[ETL]] pipelines) or continuous append-only event streams |
| **Storage Layout**     | Typically **Row-oriented** ([[B-Tree Index\|B-Trees]], [[LSM Tree\|LSM-Trees]])            | Typically **[[Column-Oriented Storage\|Column-oriented]]** (Parquet, ClickHouse, Redshift)          |
| **Schema Design**      | Normalized (e.g., 3NF) to minimize duplication             | Denormalized ([[Star Schema vs Snowflake Schema\|Star or Snowflake schemas]])                               |
| **Data Scope**         | Latest, current operational state (Gigabytes to Terabytes) | Long-term historical consolidated data (Terabytes to Petabytes)        |
| **Primary Bottleneck** | Disk seek time, latency, and transactional [[Database Lock\|locks]]           | Disk bandwidth and scan throughput                                     |
| **Examples**           | [[PostgreSQL]], MySQL, [[Redis]], DynamoDB                         | Snowflake, ClickHouse, Google BigQuery, Amazon Redshift                |

> [!warning] Failure modes & trade-offs
> - **Running OLAP queries on an OLTP database:** Even on a read replica, a heavy multi-million-row scan thrashes the buffer pool, spikes CPU/I/O, and can delay [[Replication Lag|replication replay]] (causing `canceling statement due to conflict with recovery` in [[PostgreSQL]]).
> - **Data freshness gap:** Moving data from OLTP to a [[Data Warehouse]] via nightly [[ETL]] introduces hours of staleness. Use [[Change Data Capture]] (e.g., Debezium + [[Kafka]]) when near-real-time analytics are required.

## Flashcards

What is the primary hardware bottleneck of an [[OLTP]] workload versus an [[OLAP]] workload?::OLTP is bottlenecked by disk seek time, latency, and lock contention; OLAP is bottlenecked by sequential disk bandwidth and scan throughput.
^q-qduc

Why do OLTP schemas favor normalization while OLAP schemas favor denormalization ([[Star Schema vs Snowflake Schema]])?::OLTP optimizes for fast, anomaly-free small writes; OLAP optimizes for fast read scans with fewer expensive joins across billions of rows.
^q-262e

## Related

- [[OLTP]]
- [[OLAP]]
- [[Data Warehouse]]
- [[Column-Oriented Storage]]
- [[Star Schema vs Snowflake Schema]]
- [[ETL]]
- [[Change Data Capture]]
- [[CQRS]]
