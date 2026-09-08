Index is an *additional* data structure that is derived from the primary data. Many databases allow you to add and remove indexes, and this doesn't affect the contents of the database; it only affects the performance of queries. Maintaining additional structures incurs overhead, especially on writes. 

Any kind of index usually trades write performance (as we have to update the index any time the data is written) for read performance. That is why databases don't usually index everything by default, but require you to choose indexes manually, using your knowledge of the application typical query patterns.

## Hash Index

It is the common index for key-value data. Key-value stores are quite similar to the dictionary type that you can find in many programming languages, and which are usually implemented as a hash map. It is the simplest case for [[Database Log]] implementation.

### Compaction and merging

But how not to run out of space? One approach is to split the data store into segments of a certain size. Then we can do compaction and merging to save up disk space. 
- *Compaction* - for each segment leave only latest version of a specific key value.
- Merging - Merge segments, for duplicated key leave values from later segments (each subsequent segment is newer).

Segments are never modified after they have been written, so merged segment is written to a new file. This process can be done in the background thread.

Each segment now has its own in-memory hash table, mapping keys to file offsets. In
order to find the value for a key, we first check the most recent segment’s hash map;
if the key is not present we check the second-most-recent segment, and so on. The
merging process keeps the number of segments small, so lookups don’t need to check
many hash maps.

Several considerations:
- **Concurrency control** - As writes are appended to the log in a strictly sequential order, a common implementation choice is to have only one writer thread. Data file segments are append-only and otherwise immutable, so they can be read concurrently by multiple threads.
- **Partially written records** - The database may crash at any time, including halfway through appending a record to the log. Bitcask files include checksums, allowing such corrupted parts of the log to be detected and ignored.
- **Crash recovery** - If the database is restarted, the in-memory hash maps are lost. In principle, you can restore each segment’s hash map by reading the entire segment file from beginning to end and noting the offset of the most recent value for every key as you go along. However, that might take a long time if the segment files are large, which would make server restarts painful.
- **Deleting records** - If you want to delete a key and its associated value, you have to append a special deletion record to the data file (sometimes called a tombstone). When log segments are merged, the tombstone tells the merging process to discard any previous values for the deleted key.

The hash table index also has limitations: 
- The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires a lot of random access I/O, it is expensive to grow when it becomes full, and hash collisions require fiddly logic. 
- Range queries are not efficient. For example, you cannot easily scan over all keys between kitty00000 and kitty99999—you’d have to look up each key individually in the hash maps.

## SSTables and LSM-Tress

If we take format of the [[#Hash Index]] example and change it to store pairs not by the order or write, but sorting by key, we will implement the Sorted String Table, or SSTable for short. It also requires that each key only appears once within each merged segment file (compaction ensures that). SSTables have several big advantages over log segments with hash index:
1. Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is like the one used in the mergesort algorithm: you start reading the input files side by side, look at the first key in each file, copy the lowest key (according to the sort order) to the output file, and repeat. This produces a new merged segment file, also sorted by key.
2. In order to find a particular key in the file, you no longer need to keep an index of all the keys in memory. For example: say you’re looking for the key *handiwork*, but you don’t know the exact offset of that key in the segment file. However, you do know the offsets for the keys *handbag* and *handsome*, and because of the sorting you know that *handiwork* must appear between those two. This means you can jump to the offset for *handbag* and scan from there until you find *handiwork* (or not, if the key is not present in the file). You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly
3. Since read requests need to scan over several key-value pairs in the requested range anyway, it is possible to group those records into a block and compress it before writing it to disk. Each entry of the sparse in-memory index then points at the start of a compressed block. Besides saving disk space, compression also reduces the I/O bandwidth use.

To maintain the order of writes we can use in-memory data structure such as red-black trees or AVL trees. With these data structures you can insert keys in any order and read them in sorted order.

The workflow is follows:
1. When a write comes in, it is added to in-memory data structure (it is usually called a *memtable*)
2. When the memtable gets bigger then some threshold, write it out to disk as an SSTable file.The new SSTable file becomes the most recent segment of the database. While the SSTable is being written out to disk, writes can continue to a new memtable instance.
3. In orde to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment, then in the next-older segment, etc.
4. From time to time, run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values.

This scheme works very well. It only suffers from one problem: if the database crashes, the most recent writes (which are in the memtable but not yet written out to disk) are lost. In order to avoid that problem, we can keep a separate log on disk to which every write is immediately appended. That log is not in sorted order, but that doesn’t matter, because its only purpose is to restore the memtable after a crash. Every time the memtable is written out to an SSTable, the corresponding log can be discarded.