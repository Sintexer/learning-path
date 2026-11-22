---
flashcards:
  q-vjp2: { nid: 1795356720267, hash: mjt676ws, sync: mqqn22hr }
  q-6byp: { nid: 1795356720293, hash: mngymnjd, sync: 8rj2h75z }
  q-uknp: { nid: 1795356720318, hash: gmgaimyg, sync: q7i9kvnr }
  q-wbt9: { nid: 1795356720342, hash: 7ncyu499, sync: sp8zwqab }
---

An [[In-Memory Database|in-memory]] data structure store commonly used as a low-latency cache, session store, [[Rate Limiting|rate limiter]], message broker (Streams/Pub-Sub), and coordination primitive.

## Architecture & Data Structures

- **Rich native types:** Strings, Hashes, Lists, Sets, Sorted Sets (`ZSET` — skiplist + hash table), Bitmaps ([[Bitmap]]), HyperLogLogs, Geospatial indexes, and Streams.
- **Single-threaded command execution:** One main thread executes commands sequentially using an I/O multiplexing event loop (`epoll`/`kqueue`), eliminating internal lock contention and guaranteeing atomic single-command execution. Newer versions offload network I/O parsing to I/O threads while keeping command execution single-threaded.
- **Atomicity & scripting:** Supports Lua scripts and Redis Functions that run atomically on the server without interleaving other commands.
- **Persistence options:**
  - **RDB (snapshots):** Point-in-time fork-and-dump to disk. Fast restarts, compact files, but risks losing writes since the last snapshot.
  - **AOF (Append-Only File):** Logs every write command (like a [[WAL]]) with `fsync` policies (`always`, `everysec`, `no`). Better durability, larger files, periodic rewrite needed.
- **High availability & scaling:**
  - **Redis Sentinel:** Monitors primary/replica pairs ([[Leader-Based Replication]]) and automates failover.
  - **Redis Cluster:** Shards the keyspace across 16,384 hash slots for [[Horizontal scaling]].

> [!warning] Failure modes & trade-offs
> - **Single-threaded stalls:** An $O(N)$ command (`KEYS *`, `SMEMBERS` on a huge set, `DEL` on a giant key) blocks *every* client on that instance. Always use cursor-based `SCAN` and non-blocking `UNLINK`.
> - **Fork latency & copy-on-write (CoW):** RDB snapshots and AOF rewrites call `fork()`. Under heavy write load, Linux CoW can double RAM usage and trigger OOM kills if you don't leave ~50% memory headroom.
> - **Eviction surprises:** When `maxmemory` is reached, `noeviction` fails all writes; `allkeys-lru`/`allkeys-lfu` evict keys even without TTL. Misconfiguring eviction on a Redis instance shared between cache and sessions/queues causes silent data loss.
> - **Async replication & [[Distributed Lock|distributed locks]]:** Redis replicates asynchronously, so a lock acquired on a primary can be lost on failover. For correctness-critical locks (not just best-effort deduplication), pair with a [[Fencing Token]] or use a consensus store like [[etcd]].

## Flashcards

Why can a single `KEYS *` command cause an outage in Redis?::Redis executes commands on a single thread, so an $O(N)$ scan blocks all other client requests until it finishes (use `SCAN` instead).
^q-vjp2

In Redis persistence, ==RDB== takes point-in-time snapshots via `fork()`, while ==AOF== logs every write command with configurable `fsync` intervals.
^q-6byp

## Why is Redis not a safe standalone source of truth for correctness-critical distributed locks? #card

- Redis [[Leader-Based Replication|replication]] is asynchronous: a primary can acknowledge a lock and crash before replicating it to the promoted replica, allowing a second client to acquire the same lock ([[Split-Brain]]).
- Even with Redlock, GC pauses or clock jumps can violate safety unless the downstream resource validates a monotonic [[Fencing Token]].
^q-uknp

Never mix volatile cache keys and durable state (sessions, rate-limit counters, queues) in the same Redis instance with an LRU eviction policy. #card-reminder
^q-wbt9

## Related

- [[Memcached]]
- [[In-Memory Database]]
- [[Leader-Based Replication]]
- [[Distributed Lock]]
- [[Fencing Token]]
- [[Rate Limiting]]
- [[etcd]]
