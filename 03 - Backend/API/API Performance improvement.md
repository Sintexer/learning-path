## Pagination

This is a common optimization when the size of the result is large. The results are returned in bounded pages (or streamed back to the client) to improve service responsiveness and cap memory usage.

- **Offset pagination (`LIMIT / OFFSET`):** Simple and supports jumping to page N, but degrades to O(offset) on deep pages and suffers from skipped/duplicated rows when concurrent writes occur.
- **Cursor / keyset pagination (`WHERE id > last_id LIMIT N`):** Uses an index seek (`O(log N)`), stays fast at any depth, and remains stable under concurrent inserts.

## Asynchronous Logging

Synchronous logging deals with the disk for every call and can slow down the system. Asynchronous logging sends logs to a lock-free buffer first and immediately returns. The logs will be flushed to the disk periodically. This significantly reduces the I/O overhead.

## Caching

We can store frequently accessed data into a cache. The client can query the cache first instead of visiting the [[Database|database]] directly. If there is a cache miss, the client can query from the database. Caches like [[Redis]] or [[Memcached]] store data in memory, so data access is much faster than the database (and static/public responses can be cached even closer at a [[CDN]]).

## Payload Compression

Requests and responses can be compressed (e.g. gzip or Brotli over [[HTTP2]]) so that the transmitted data size is much smaller. This speeds up upload and download over the network.

## Connection Pool

When accessing resources, we often need to load data from the [[Database|database]]. Opening and closing DB connections adds significant overhead (TCP + [[TLS]] handshake, auth, session allocation). So we should connect to the DB via a pool of open connections. The connection pool is responsible for managing the connection lifecycle.

> [!warning] Failure modes & trade-offs
> - **Cache stampede (thundering herd):** When a hot key expires, hundreds of concurrent requests miss the cache simultaneously and hammer the DB. Mitigate with request coalescing (singleflight), probabilistic early expiration, or jittered TTLs.
> - **Oversized DB connection pools:** Larger pools do not mean higher throughput—too many active connections cause DB lock/context-switch contention. Size pools conservatively using [[Little's Law]] and DB core count (often `~2× CPU cores` per DB instance, shared across app replicas).
> - **Async logging loss on crash:** Unflushed ring-buffer entries can be lost if the JVM crashes or gets OOMKilled; configure drop vs. block policies carefully so a full log buffer never stalls request threads.
> - **Compression CPU trade-off:** Compressing tiny payloads (<1 KB) or already-compressed binary data wastes CPU for negligible bandwidth savings.

## Flashcards

## Why does cursor (keyset) pagination outperform `OFFSET` pagination on large tables? #card
`OFFSET N` forces the database to scan and discard `N` rows on every page (linear degradation as pages get deeper) and can skip or duplicate rows if items are inserted/deleted between requests. Cursor pagination seeks directly via an index (`WHERE id > :cursor LIMIT k`) in `O(log N)` time with stable ordering.

What is a cache stampede (thundering herd), and name two mitigations?::Many concurrent requests miss an expired hot cache key at once and overload the backing database; mitigate with singleflight/lock coalescing, jittered TTLs, or probabilistic early refresh.

Do not oversize database connection pools to match HTTP thread counts—keep pools small and bounded to prevent database contention, and apply [[Backpressure]] or [[Rate Limiting]] upstream. #card-reminder

## Related
- [[API]]
- [[Redis]]
- [[CDN]]
- [[HTTP2]]
- [[Little's Law]]
- [[Backpressure]]
- [[Rate Limiting]]
- [[persistence]]