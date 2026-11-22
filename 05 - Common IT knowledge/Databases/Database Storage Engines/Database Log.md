Databases usually use a *log*, which is different from an application logger. A **database log** is an append-only file that achieves high write throughput because sequential disk writes avoid random seeks. Databases using append-only logs must also handle concurrency control, reclaiming disk space so that the log does not grow forever (via **compaction** and segment merging), and handling crash errors and partially written records (via checksums).

## Simplest case

The simplest example is a running append-only file for key-value data. Each new write is appended to the end of the file. To support quick data access without scanning the entire file, it pairs the log with an in-memory [[Hash Index|hash index]] mapping each key to its latest byte offset in the log (the architecture used by **Bitcask**, the default storage engine of Riak).

## Compaction and Segment Merging

Because an append-only log grows indefinitely, the engine breaks it into fixed-size **segments**:
1. When a segment reaches a threshold size, it is closed and new writes go to a fresh segment file.
2. **Compaction** discards duplicate keys in closed segments, keeping only the most recent update for each key.
3. Multiple compacted segments are merged in the background into a new file, and reads atomically switch over to the merged segment while old files are deleted.

> [!warning] Failure modes & trade-offs
> - **Hash-index limitations**: An append-only log + in-memory [[Hash Index]] requires all keys to fit in RAM and cannot perform efficient range queries (`WHERE key BETWEEN a AND b`). Solving both limitations leads directly to [[SSTable]]s and [[LSM Tree]]s.
> - **Deletes require tombstones**: You cannot overwrite or erase an old record in an immutable segment; deletes append a special tombstone record that compaction later discards.
> - **Crash recovery of in-memory index**: Rebuilding an in-memory hash map from a huge log on restart is slow; engines periodically snapshot the hash map to disk.

What is the difference between a database log and an application log?::A database log is a structured, append-only binary sequence of records designed for storage durability, crash recovery ([[WAL]]), or [[Replication Log|replication]], whereas an application log is human-readable diagnostic output.

## What are the two main limitations of an append-only log paired with an in-memory hash index (Bitcask)? #card
1. All keys must fit in RAM (large key spaces cannot be indexed on disk efficiently with a hash map).
2. Range queries are inefficient because keys are not stored in sorted order (you must look up every key individually).

In log-structured storage, ==compaction== reclaims disk space by discarding overwritten keys and tombstones across closed segments while preserving only the latest value per key.

## Related
- [[WAL]]
- [[Database Storage Engines]]
- [[Hash Index]]
- [[SSTable]]
- [[LSM Tree]]
- [[Replication Log]]
- [[DDIA Chapter 3 Storage and Retrieval]]
