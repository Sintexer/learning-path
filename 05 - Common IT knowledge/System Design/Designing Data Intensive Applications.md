[[Reliability, Scalability, Maintainability]]
## Chapter 2: Data models

- [[Relational Database]]
- [[Document Database]]
- [[SQL vs NoSQL]]
- [[Query Languages for Data]]
- [[MapReduce querying]]
- [[Graph database]]

## Chapter 3: Storage and Retrieval


Databases handle data storage and retrieval using two main approaches optimized for different workloads: **OLTP** (transaction processing) and **OLAP** (analytics).

### OLTP (Online Transaction Processing)

**Use case:** User-facing applications (banking, e-commerce, social media)

- High volume of requests
- Each query touches a small number of records
- Access pattern: key-based lookups
- **Bottleneck:** Disk seek time
- **Optimization:** Indexes to find specific data quickly

### OLAP (Online Analytical Processing)

**Use case:** Data warehouses, business analytics

- Low volume of queries (but each is demanding)
- Scans millions of records per query
- Access pattern: sequential scans across large datasets
- **Bottleneck:** Disk bandwidth
- **Optimization:** Column-oriented storage to minimize data read from disk


### OLTP Storage Engines: Two Schools

#### 1. Log-Structured

Never updates existing files; only appends and deletes obsolete files.

**Key idea:** Convert random-access writes into sequential writes for higher throughput.

**Examples:** Bitcask, SSTables, LSM-trees, LevelDB, Cassandra, HBase, Lucene

#### 2. Update-in-Place

Treats disk as fixed-size pages that can be overwritten.

**Examples:** B-trees (used in most relational and many nonrelational databases)

### OLAP Optimization: Column-Oriented Storage

Traditional row-oriented databases store entire rows together. Column-oriented storage stores each column separately, enabling:

- Compact encoding of data
- Reading only the columns needed for a query
- Significant speed improvements for analytical workloads
### Key Takeaways

|Aspect|OLTP|OLAP|
|---|---|---|
|**Workload**|Many small, fast queries|Few large, complex queries|
|**Data access**|Key-based lookups|Sequential scans|
|**Critical resource**|Disk seek time|Disk bandwidth|
|**Storage focus**|Indexes, fast writes|Compression, read efficiency|
|**Best storage engine**|B-trees or log-structured|Column-oriented|

- [[Database Log]]
- [[Database Index]]
- [[Database Index Practical Usage]]
- [[Transaction]]
- [[OLTP and OLAP]]
- [[Star Schema vs Snowflake Schema]]
- [[In-Memory Database]]
- [[Data Warehouse]]