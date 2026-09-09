

In the context of [[Data Warehouse]] and [[OLAP]], If you have trillions of rows and petabytes of data in your fact tables, storing and querying them efficiently becomes a challenging problem. Dimension tables are usually much smaller (millions of rows), so in this section we will concentrate primarily on storage of facts.

Although fact tables are often over 100 columns wide, a typical data warehouse query only accesses 4 or 5 of them at one time ("SELECT *" queries are rarely needed for analytics).

In most [[OLTP]] databases, storage is laid out in a row-oriented fashion: all the values from one row of a table are stored next to each other. Document databases are similar: an entire document is typically stored as one contiguous sequence of bytes.

Using indexes for such purpose won't solve the problem: DB would still need to load all 100 column values for each row, and filter them in memory. That cam take a long time.

The idea behind column-oriented storage is simple: don’t store all the values from one row together, but store all the values from each column together instead. If each column is stored in a separate file, a query only needs to read and parse those columns that are used in that query, which can save a lot of work.

> [!tip]
> Column storage is easiest to understand in a relational data model, but it applies equally to nonrelational data. For example, Parquet is a columnar storage format that supports a document data model, based on Google’s Dremel

The column-oriented storage layout relies on each column file containing the rows in the same order. Thus, if you need to reassemble an entire row, you can take the 23rd entry from each of the individual column files and put them together to form the 23rd row of the table.

## Column Compression

Besides only loading those columns from disk that are required for a query, we can further reduce the demands on disk throughput by compressing data. Fortunately, column-oriented storage often lends itself very well to compression.

Often, the number of distinct values in a column is small compared to the number of rows (for example, a retailer may have billions of sales transactions, but only 100,000 distinct products). We can now take a column with n distinct values and turn it into n separate bitmaps: one bitmap for each distinct value, with one bit for each row. The bit is 1 if the row has that value, and 0 if not.

If n is very small (for example, a country column may have approximately 200 dis‐ tinct values), those bitmaps can be stored with one bit per row. But if n is bigger, there will be a lot of zeros in most of the bitmaps (we say that they are sparse). In that case, the bitmaps can additionally be run-length encoded.

> [!warning] Column-oriented storage and column families
>  Cassandra and HBase have a concept of column families, which they inherited from Bigtable. However, it is very misleading to call them column-oriented: within each column family, they store all columns from a row together, along with a row key, and they do not use column compression. Thus, the Bigtable model is still mostly row-oriented.

The column oriented storages also benefit from optimized CPU L1 cache and SIMD instructions.

## Sorting

Usually column-oriented storages are sorted by the update time. However it is possible to impose order. It is not possible to sort columns apart from each other due to the architecture. but it is possible to sort by one, two, or more fields.

The administrator of the database can choose the columns by which the table should be sorted, using their knowledge of common queries. For example, if queries often target date ranges, such as the last month, it might make sense to make date_key the first sort key. Then the query optimizer can scan only the rows from the last month, which will be much faster than scanning all rows.

Another advantage of sorted order is that it can help with compression of columns. If the primary sort column does not have many distinct values, then after sorting, it will have long sequences where the same value is repeated many times in a row. A simple run-length encoding could compress that column down to a few kilobytes—even if the table has billions of rows.

## Writing to column-oriented storage

These optimizations make sense in data warehouses, because most of the load consists of large read-only queries run by analysts. Column-oriented storage, compression, and sorting all help to make those read queries faster. However, they have the downside of making writes more difficult. 

An update-in-place approach, like B-trees use, is not possible with compressed columns. If you wanted to insert a row in the middle of a sorted table, you would most likely have to rewrite all the column files. As rows are identified by their position within a column, the insertion has to update all columns consistently.

Fortunately, we have already seen a good solution earlier in this chapter: LSM-trees. All writes first go to an in-memory store, where they are added to a sorted structure and prepared for writing to disk. It doesn’t matter whether the in-memory store is row-oriented or column-oriented. When enough writes have accumulated, they are merged with the column files on disk and written to new files in bulk. This is essentially what Vertica does.

Queries need to examine both the column data on disk and the recent writes in memory, and combine the two. However, the query optimizer hides this distinction from the user. From an analyst’s point of view, data that has been modified with inserts, updates, or deletes is immediately reflected in subsequent queries.

## Aggregation: Data Cubes and Materialized Views

**Columnar Storage:**
- Faster than traditional row-oriented databases for analytical queries
- Rapidly gaining adoption in data warehouses

**Materialized Views & Data Cubes:**
- **Materialized view**: A cached copy of query results stored on disk (vs. virtual views which recompute on-the-fly)
- **Data cube (OLAP cube)**: A special materialized view—a multidimensional grid of pre-aggregated data (SUM, COUNT, etc.) grouped by different dimensions

**Example:** A 2D cube has dates on one axis, products on the other; each cell contains total sales for that date-product combination. A 5D cube would have date, product, store, promotion, and customer dimensions.

**Trade-offs:**

|Advantage|Disadvantage|
|---|---|
|Precomputed aggregates = very fast queries|Less flexible—can't compute metrics using non-dimensions (e.g., "% of sales from items >$100")|
|Cuts millions of rows to one lookup|Maintenance overhead (must update when raw data changes)|
|Makes sense in read-heavy warehouses|Not ideal for OLTP (write-heavy) systems|

**Best practice:** Keep raw data in the warehouse; use data cubes/aggregates only as a performance optimization for frequently-run queries, not as the primary data store.