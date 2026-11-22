---
flashcards:
  q-6htc: { nid: 1795356728770, hash: r3qud8us, sync: 74zx4cq9 }
  q-vwbt: { nid: 1795356728803, hash: jfpvgfuf, sync: cdz54wwq }
  q-282h: { nid: 1795356728818, hash: vqjajg5t, sync: 3v6phieu }
---

**ETL (Extract, Transform, Load)** is the automated pipeline used to move data from operational [[OLTP]] databases into an analytical [[Data Warehouse]] ([[OLAP]]):

1. **Extract:** Pull raw data from multiple disparate sources (databases, logs, APIs, event streams).
2. **Transform:** Clean, standardize, join, and restructure the data for analysis (e.g., deduplicating, converting currencies, normalizing timestamps, building [[Star Schema vs Snowflake Schema|fact/dimension tables]]).
3. **Load:** Write the finalized data into a target [[Data Warehouse]] for querying and business intelligence.

## ETL vs. ELT

- **ETL (traditional):** Transforms data in a dedicated compute tier *before* loading into the warehouse. Useful when PII must be stripped before landing in the warehouse or when warehouse compute is expensive.
- **ELT (modern cloud warehouses):** Loads raw data directly into cheap cloud storage / warehouse staging tables first, then transforms it in-place using SQL (e.g., dbt on Snowflake, BigQuery, Redshift). Preserves raw data for future re-processing without re-extracting from source systems.

## Extraction Patterns

- **Full dump:** Simple, but does not scale beyond small lookup tables.
- **Incremental polling (`updated_at > last_cursor`):** Misses hard `DELETE`s and risks skew if `updated_at` is set by application clocks rather than DB commit time.
- **[[Change Data Capture]] (CDC):** Tails the database [[WAL]] / binlog (e.g., Debezium → [[Kafka]]) for low-latency, lossless capture of inserts, updates, and deletes without querying the OLTP tables directly.

> [!warning] Failure modes & trade-offs
> - **Non-idempotent loads cause duplicate metrics:** If a batch job crashes halfway and retries, naive `INSERT` appends duplicate rows and inflates revenue reports. Design every load stage to be [[Idempotency|idempotent]] (partition overwrite, `MERGE`/upsert, or atomic table swap).
> - **Late-arriving data & backfills:** Events can arrive hours late; watermarks and re-runnable date-partitioned pipelines are essential.
> - **Upstream schema breaking changes:** An OLTP team renaming a column breaks downstream ETL unless protected by contracts or a staging layer.

## Flashcards

What is the key architectural difference between **ETL** and **ELT**?::ETL transforms data in an external pipeline *before* loading into the [[Data Warehouse]]; ELT loads raw data into the warehouse first and transforms it in-place (preserving raw history for replay).
^q-6htc

Why is [[Change Data Capture]] (CDC) preferred over polling `WHERE updated_at > ?` for extracting data from [[OLTP]] databases?::CDC reads the database's [[WAL]]/binlog directly, capturing hard deletes without missing rows or adding heavy query load to the OLTP database.
^q-vwbt

Make every ETL/ELT load stage idempotent (via partition overwrite or upsert) so retries and backfills never double-count metrics. #card-reminder
^q-282h

## Related

- [[Data Warehouse]]
- [[OLAP]]
- [[OLTP and OLAP]]
- [[Star Schema vs Snowflake Schema]]
- [[Change Data Capture]]
- [[Idempotency]]
- [[Kafka]]
