---
flashcards:
  q-7var: { nid: 1795356728042, hash: wfascqnz, sync: kkecvwgd }
  q-nvna: { nid: 1795356728067, hash: abuiyaga, sync: 3bjqmfqy }
---

It is the common index for key-value data. Key-value stores are quite similar to the dictionary type that you can find in many programming languages, and which are usually implemented as a hash map. It is the simplest case for [[Database Log]] implementation.

### The Internal Mechanics

- Computes an internal 32-bit hash of the indexed value.
- Places the hash into a specific **bucket page**.
- The bucket stores the hash value and the pointer (TID) to the heap tuple.

### When to use it vs. B-Tree?

- **Only for strict equality lookups (`=`)**.
- **Time Complexity:** Average O(1) page access compared to O(3–4) for [[B-Tree Index]].

### Why interviewers rarely recommend it:

- It **cannot** do range scans (`>`, `<`).
- It **cannot** optimize `ORDER BY`.
- It **cannot** do partial key matching on composite columns.
- In practice, the 3–4 pages of a B-tree lookup are usually cached in RAM anyway (in PostgreSQL's `shared_buffers` or the OS page cache). Thus, the negligible speedup of Hash rarely outweighs its lack of flexibility.


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

> [!tip] Production reality
> - **[[PostgreSQL]] Hash indexes (PG 10+):** Modern PostgreSQL hash indexes are [[WAL]]-logged and crash-safe (older versions were not!). Still, they only beat [[B-Tree Index|B-Trees]] in niche cases like indexing very long strings (e.g., URLs) for exact equality where storing only the 32-bit hash makes the index much smaller than a B-Tree.
> - **Bitcask-style hash indexes:** Fast for point lookups and restarts with snapshots (e.g., Riak), but when keys outgrow RAM or range queries are needed, storage engines move to [[SSTable]] + [[LSM Tree]].

## Flashcards

Why are [[Hash Index|hash indexes]] rarely chosen over [[B-Tree Index|B-Tree indexes]] in relational databases?::Hash indexes only support strict equality (`=`) — no range scans, `ORDER BY`, or composite prefix lookups — and a B-Tree's top 3–4 levels are already cached in RAM anyway.
^q-7var

In a log-structured hash index (Bitcask), how are deletes handled on immutable append-only segments?::By appending a special deletion marker called a **tombstone**, which tells background segment compaction/merging to discard earlier versions of that key.
^q-nvna

## Related

- [[Database Index]]
- [[B-Tree Index]]
- [[LSM Tree]]
- [[SSTable]]
- [[Database Log]]
- [[PostgreSQL]]
- [[DDIA Chapter 3 Storage and Retrieval]]
