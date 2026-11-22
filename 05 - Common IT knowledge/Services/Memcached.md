A high-performance, distributed, [[In-Memory Database|in-memory]] key-value cache designed for sub-millisecond read offloading in front of primary [[Database|databases]].

## Core Characteristics

- **Pure in-memory volatile cache:** Data is never persisted to disk. A node restart starts with a cold cache.
- **Simple data model:** Stores opaque byte blobs (`key -> value`) with optional TTL and atomic `incr`/`decr` and `cas` ([[Compare-And-Swap]]) operations. Max item size defaults to 1 MB.
- **Slab allocator:** Pre-allocates memory into fixed-size chunk classes (slabs) to avoid heap fragmentation. Evicts least recently used (LRU) items within the slab class when full.
- **Multi-threaded architecture:** Unlike [[Redis]]'s single-threaded command loop, Memcached scales vertically across multiple CPU cores on a single host.
- **Client-side sharding:** Servers are completely unaware of each other. Clients route keys across nodes using [[Consistent Hashing]] so adding/removing a node invalidates only $\approx 1/N$ of keys.

> [!warning] Failure modes & trade-offs
> - **Thundering herd (cache stampede):** When a hot key expires or a node restarts, thousands of concurrent requests miss the cache and hammer the origin database. Mitigate with request coalescing (singleflight), probabilistic early expiration, or jittered TTLs.
> - **Slab calcification:** If workload object sizes change over time, memory allocated to old slab classes may sit underutilized while another class evicts aggressively (mitigated by slab automove).
> - **No replication or failover:** Because nodes are independent, losing a node means cache misses for that shard until the cache warms up again.

## Memcached vs. Redis

- Choose **Memcached** when you need a simple, multi-threaded, low-overhead cache for small opaque values and want maximum single-node CPU utilization.
- Choose **[[Redis]]** when you need rich data structures (sorted sets, hashes, streams), persistence, [[Replication]], Lua scripting, or pub/sub.

## Flashcards

How does Memcached distribute keys across a cluster of nodes?::Through client-side [[Consistent Hashing]]; Memcached servers do not communicate with or know about one another.

Memcached avoids memory fragmentation using a ==slab allocator==, which can suffer from ==slab calcification== when object size distributions shift over time.

When should you prefer Memcached over [[Redis]]?::When you only need a simple volatile key-value cache and want multi-threaded vertical scaling across CPU cores with minimal memory overhead.

## Related

- [[Redis]]
- [[In-Memory Database]]
- [[Consistent Hashing]]
- [[Compare-And-Swap]]
- [[Horizontal scaling]]
