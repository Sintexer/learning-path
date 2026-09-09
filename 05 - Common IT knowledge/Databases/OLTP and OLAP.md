- [[OLTP]]
- [[OLAP]]

| Property               | OLTP (Online Transaction Processing)                       | OLAP (Online Analytical Processing)                                    |
| :--------------------- | :--------------------------------------------------------- | :--------------------------------------------------------------------- |
| **Primary Use Case**   | Day-to-day operations, web apps, user interactions         | Business intelligence, reporting, data science                         |
| **Read Pattern**       | Small number of records per query, fetched by key          | Scans millions of rows over a few specific columns                     |
| **Write Pattern**      | Frequent, random writes and updates from end-users         | Bulk loads ([[ETL]] pipelines) or continuous append-only event streams |
| **Storage Layout**     | Typically **Row-oriented** (B-Trees, LSM-Trees)            | Typically **Column-oriented** (Parquet, ClickHouse, Redshift)          |
| **Schema Design**      | Normalized (e.g., 3NF) to minimize duplication             | Denormalized (Star or Snowflake schemas)                               |
| **Data Scope**         | Latest, current operational state (Gigabytes to Terabytes) | Long-term historical consolidated data (Terabytes to Petabytes)        |
| **Primary Bottleneck** | Disk seek time, latency, and transactional locks           | Disk bandwidth and scan throughput                                     |
| **Examples**           | PostgreSQL, MySQL, Redis, DynamoDB                         | Snowflake, ClickHouse, Google BigQuery, Amazon Redshift                |
