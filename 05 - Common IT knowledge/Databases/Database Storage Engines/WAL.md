---
flashcards:
  q-ayan: { nid: 1795356726592, hash: zy5t5cdi, sync: h67i3r2b }
---

**WAL (Write-Ahead Log)** — also known as a *redo log* — is an append-only [[Database Log|log file]] to which every storage modification must be durably written **before** the corresponding in-memory or on-disk data pages (e.g., [[B-Tree Index|B-Tree]] pages or an [[LSM Tree#Memtable|LSM-Tree memtable]]) are updated.

## Why WAL Exists

1. **Crash recovery (Atomicity & Durability in [[ACID]])**:
   - In **[[B-Tree Index|B-Tree]] engines**, splitting a page requires writing multiple pages to disk. If the machine crashes mid-split, the tree would have dangling pointers and corrupted structure. On restart, the engine replays the WAL from the last **checkpoint** to restore the B-Tree to a consistent state.
   - In **[[LSM Tree]] engines**, writes are buffered in a volatile in-memory memtable. On crash, the WAL is replayed to reconstruct the lost memtable before flushing it to an [[SSTable]].
2. **Sequential I/O performance**: Appending to a WAL is a single sequential disk write (and `fsync`), allowing the database to acknowledge a `COMMIT` without waiting for random writes to scattered data pages (which are flushed lazily from the buffer pool in the background).
3. **[[Replication Log#Write-ahead log (WAL) shipping|WAL shipping & Replication]]**: Engines like [[PostgreSQL]] stream WAL segments to followers for [[Leader-Based Replication]] and point-in-time recovery (PITR).

## Checkpointing & Truncation

Because a WAL cannot grow forever, the engine periodically performs a **checkpoint**: flushing all dirty pages from the buffer pool to disk (or flushing a memtable to an [[SSTable]]). Once the checkpoint completes, older WAL segments prior to the checkpoint LSN (Log Sequence Number) can be archived or reclaimed.

> [!warning] Failure modes & trade-offs
> - **`fsync` vs throughput (`group commit`)**: Calling `fsync()` on every single transaction limits throughput to disk IOPS. Production engines batch concurrent commits into a single `fsync()` (**group commit**). Disabling `fsync` or using `synchronous_commit = off` trades durability (risking recent committed writes on power loss) for lower latency.
> - **Torn writes & checksums**: A power failure mid-write can leave a partially written WAL record. Engines guard every WAL record with CRC checksums and discard incomplete trailing records on recovery.
> - **Checkpoint storms**: Large buffer pools with infrequent checkpoints cause huge bursts of dirty-page disk I/O during checkpoints and slow crash recovery times (more WAL to replay).
> - **Write amplification**: Every mutation is written at least twice — once to the WAL and later to the actual data pages/SSTables (plus full-page writes in [[PostgreSQL]] after a checkpoint to guard against torn 8 KB pages).

## Why does a B-Tree storage engine need a Write-Ahead Log (WAL)? #card
Modifying a [[B-Tree Index|B-Tree]] (especially a page split) requires overwriting multiple disk pages. If the database crashes halfway through, the tree structure can be left corrupted. Recording every modification in an append-only WAL first lets the engine replay the log after a crash to restore consistency and [[ACID]] durability.

What happens to old WAL segments once the storage engine completes a checkpoint?::Dirty pages (or memtables) up to the checkpoint LSN are safely on disk, so older WAL segments can be archived or truncated to reclaim disk space.

In high-throughput databases, ==group commit== amortizes the cost of `fsync()` by flushing the WAL records of multiple concurrent transactions in a single disk sync.
^q-ayan

## Related
- [[Database Log]]
- [[Database Storage Engines]]
- [[B-Tree Index]]
- [[LSM Tree]]
- [[Replication Log]]
- [[PostgreSQL]]
- [[ACID]]
