
Falling RAM costs have made it practical to store entire datasets in memory, giving rise to systems that either treat data as volatile caches (e.g., [[Memcached]]) or maintain durability through append-only disk logs ([[WAL]]), snapshots, or [[Replication|replication]] (e.g., VoltDB, [[Redis]], RAMCloud). 

For durable in-memory engines, disk is used strictly for crash recovery and backups while reads are served entirely from memory. Counterintuitively, their primary performance advantage is not merely avoiding disk reads — since traditional databases can already keep hot working sets in the operating system's disk page cache — but eliminating the substantial CPU overhead of serializing and managing complex on-disk page layouts, [[Database Latch|latches]], and buffer pools. Furthermore, keeping data purely in RAM makes it simpler to implement richer, non-relational data structures like queues and sets (as seen in [[Redis]]). 

When memory runs out, emerging designs employ "anti-caching" to evict least-recently-used records to disk at the granularity of individual items rather than coarse OS memory pages, while the ongoing evolution of non-volatile memory (NVM) promises to shift these [[Database Storage Engines|storage engine]] boundaries even further.

> [!warning] Failure modes & trade-offs
> - **Hard RAM ceiling & OOM cliff:** Unlike disk-backed engines that degrade gracefully when the working set exceeds RAM, in-memory stores hit a wall (eviction storms or rejected writes) as soon as dataset size approaches available memory.
> - **Cold-start recovery time:** Reloading a 100 GB dataset from an RDB snapshot or replaying an AOF log after a crash can take minutes, during which the node cannot serve traffic.
> - **Cost per GB:** DRAM remains an order of magnitude more expensive than NVMe SSDs; keep hot/ephemeral state in RAM and durable cold state on disk.

## Flashcards

What is the *true* primary performance advantage of an [[In-Memory Database]] over a disk-backed database whose hot dataset already fits in OS page cache?::Eliminating the CPU overhead of encoding/decoding on-disk page formats, buffer-pool management, and page [[Database Latch|latches]] — plus enabling native in-memory data structures (like [[Redis]] sorted sets and queues).

How do durable [[In-Memory Database|in-memory databases]] survive power loss or crashes?::By writing an append-only log ([[WAL]]/AOF), taking periodic snapshots to disk, and/or replicating writes to other nodes in memory.

## Related

- [[Redis]]
- [[Memcached]]
- [[Database Storage Engines]]
- [[WAL]]
- [[DDIA Chapter 3 Storage and Retrieval]]