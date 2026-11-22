A widely used [[Document Database]] that stores semi-structured records as **BSON** (Binary JSON) documents grouped into collections.

## Architecture & Core Features

- **Flexible schema (schema-on-read):** Documents in the same collection can have different fields, reducing [[Impedance mismatch]] with application objects and supporting rapid iteration.
- **WiredTiger storage engine:** Uses a [[B-Tree Index|B-Tree]] structure with document-level concurrency control, compression, and a [[WAL|write-ahead journal]] for durability.
- **Indexing:** Supports single-field, compound, multikey (array), text, geospatial, and TTL [[Database Index|indexes]].
- **[[Replication]] (Replica Sets):** Uses [[Leader-Based Replication]] with automatic failover via a [[Raft]]-like election protocol. Writes go to the primary and propagate via the `oplog` (a capped [[Replication Log]]).
- **[[Data partitioning|Sharding]]:** Horizontally partitions large collections across shards using a **shard key** (hashed or range-based), routed through `mongos` routers and config servers.
- **Transactions:** Supports multi-document [[ACID]] [[Transaction|transactions]] with [[Snapshot Isolation]], though single-document atomic updates remain the idiomatic fast path.

> [!warning] Failure modes & trade-offs
> - **Unbounded document growth:** Pushing events into an array inside a document (e.g., comments or logs) hits the 16 MB BSON limit and causes expensive document relocations and memory pressure. Model unbounded 1-to-many relations as separate collections.
> - **Bad shard key choice:** A monotonically increasing shard key (like `ObjectId` or timestamp) with range sharding sends all writes to a single hot shard. Shard keys are costly to change later.
> - **Write concern vs. durability:** Default `w: 1` acknowledges after the primary writes to memory; if the primary crashes before replicating, writes can be rolled back. Use `w: "majority"` and `readConcern: "majority"` (or `"linearizable"`) for critical financial/stateful data.
> - **Multi-document transaction overhead:** Cross-shard transactions rely on [[Two-Phase Commit]] and significantly reduce throughput compared to single-document atomicity.

## Flashcards

What is the primary replication mechanism in MongoDB?::Replica sets using [[Leader-Based Replication]], where the primary records mutations in an `oplog` that secondaries replay asynchronously.

In MongoDB, using a monotonically increasing shard key with range sharding causes ==all new writes to hit a single hot shard==.

## What write/read concerns are needed in MongoDB to avoid losing acknowledged writes on primary failover? #card

- Set `writeConcern: { w: "majority", j: true }` so writes are journaled on a majority of replica set members before acknowledgement.
- Pair with `readConcern: "majority"` (or `"linearizable"` when strict [[Linearizability]] is required) so reads do not observe uncommitted data that could later roll back.

Model MongoDB documents around access patterns, keep arrays bounded well below the 16 MB limit, and rely on single-document atomicity whenever possible. #card-reminder

## Related

- [[Document Database]]
- [[NoSQL database]]
- [[SQL vs NoSQL]]
- [[Leader-Based Replication]]
- [[Data partitioning]]
- [[Impedance mismatch]]
- [[Snapshot Isolation]]
- [[Two-Phase Commit]]
