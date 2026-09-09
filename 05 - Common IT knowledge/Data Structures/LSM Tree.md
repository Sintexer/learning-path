> Log-Structured Merge-Tree

An **LSM-Tree (Log-Structured Merge-Tree)** is a data structure optimized for write-heavy workloads, commonly used in modern [[NoSQL database]]s, key-value stores, and distributed storage engines (such as Apache Cassandra, RocksDB, LevelDB, and ClickHouse). It is essentially an implementation of [[Database Index]].

Traditional databases (like traditional relational databases using B-Trees) update data in place on disk. When a record changes, the database seeks to the specific disk page containing that record and overwrites it. While great for reads, random disk writes are slow because of disk head movement and seek times (or frequent page-level writes on SSDs).

An **LSM-Tree** flips this approach by **turning random writes into sequential writes**. Instead of updating data in place, it buffers writes in memory and appends them sequentially to disk files, achieving massive write throughput.

SSTables have several big advantages over database log segments with hash index:
1. Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is like the one used in the mergesort algorithm: you start reading the input files side by side, look at the first key in each file, copy the lowest key (according to the sort order) to the output file, and repeat. This produces a new merged segment file, also sorted by key.
2. In order to find a particular key in the file, you no longer need to keep an index of all the keys in memory. For example: say you’re looking for the key *handiwork*, but you don’t know the exact offset of that key in the segment file. However, you do know the offsets for the keys *handbag* and *handsome*, and because of the sorting you know that *handiwork* must appear between those two. This means you can jump to the offset for *handbag* and scan from there until you find *handiwork* (or not, if the key is not present in the file). You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly
3. Since read requests need to scan over several key-value pairs in the requested range anyway, it is possible to group those records into a block and compress it before writing it to disk. Each entry of the sparse in-memory index then points at the start of a compressed block. Besides saving disk space, compression also reduces the I/O bandwidth use.

To maintain the order of writes we can use in-memory data structure such as red-black trees or AVL trees. With these data structures you can insert keys in any order and read them in sorted order.

The workflow is follows:
1. When a write comes in, it is added to in-memory data structure (it is usually called a *memtable*)
2. When the memtable gets bigger then some threshold, write it out to disk as an SSTable file.The new SSTable file becomes the most recent segment of the database. While the SSTable is being written out to disk, writes can continue to a new memtable instance.
3. In order to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment, then in the next-older segment, etc.
4. From time to time, run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values.

## Memtable

When data (inserts, updates, or deletes) comes into the database, it is first written to an in-memory data structure (typically a skip list or a self-balancing binary search tree like a Red-Black tree) that keeps keys sorted. Because it is in memory, writes are extremely fast. A **Write-Ahead Log (WAL)** is also written to disk simultaneously to ensure durability in case of a crash.

Once the active Memtable reaches a certain size threshold (e.g., a few megabytes or gigabytes), it becomes "immutable," and a new active Memtable is created to handle incoming writes.

## Flushing to disk

A background worker thread takes the immutable Memtable and flushes its sorted contents sequentially to disk. This flushed file is called an [[SSTable]].

## Usage of log

This scheme works very well. It only suffers from one problem: if the database crashes, the most recent writes (which are in the memtable but not yet written out to disk) are lost. In order to avoid that problem, we can keep a separate [[Database Log|log]] on disk to which every write is immediately appended. That log is not in sorted order, but that doesn’t matter, because its only purpose is to restore the memtable after a crash. Every time the memtable is written out to an SSTable, the corresponding log can be discarded.

## Performance optimizations

This scheme might perform bad on seeks for keys that don't exist in the DB. To solve this, storage engines usually use [[Bloom Filter]]s.

There are also different strategies to determine the order and timing of how SSTables are compacted and merged. The most common options are size-tiered and leveled compaction. LevelDB and RocksDB use leveled compaction (hence the name of LevelDB), HBase uses size-tiered, and Cassandra supports both. In leveled compaction, the key range is split up into smaller SSTables and older data is moved into separate “levels,” which allows the compaction to proceed more incrementally and use less disk space.

Even though there are many subtleties, the basic idea of LSM-trees—keeping a cascade of SSTables that are merged in the background—is simple and effective. Even when the dataset is much bigger than the available memory it continues to work well. Since data is stored in sorted order, you can efficiently perform range queries (scanning all keys above some minimum and up to some maximum), and because the disk writes are sequential the LSM-tree can support remarkably high write throughput.