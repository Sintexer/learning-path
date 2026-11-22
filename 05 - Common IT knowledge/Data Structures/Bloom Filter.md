A **Bloom filter** is a space-efficient, probabilistic [[Bitmap|bit-array]] data structure for testing set membership. It answers one question in $O(k)$ time:
- **"Definitely NOT in the set"** — 100% accurate (zero false negatives).
- **"Possibly in the set"** — small, tunable probability of a false positive.

## How It Works

1. Allocate a bit array of $m$ bits, all initialized to `0`, and pick $k$ independent hash functions.
2. **Insert key**: Hash the key with all $k$ hash functions and set those $k$ bit positions to `1`.
3. **Query key**: Hash the key with the same $k$ hash functions.
   - If *any* of the $k$ bits is `0`, the key was **definitely never inserted**.
   - If *all* $k$ bits are `1`, the key is **probably present** (or those bits were set by collisions from other keys).

## Where It Is Used in Production

- **[[LSM Tree]] storage engines** (RocksDB, LevelDB, [[Apache Cassandra]]): Each [[SSTable]] stores a Bloom filter in memory. When looking up a nonexistent key, the engine checks the Bloom filter first and skips disk I/O for SSTables that cannot contain the key.
- **[[CDN]]s & Web caches**: Avoid caching "one-hit wonders" by only writing a URL to disk cache on its *second* request (tracked via a Bloom filter).
- **Distributed databases & routers**: Quickly check whether a partition or node holds a key before sending a network RPC.

> [!warning] Failure modes & trade-offs
> - **No deletions in standard Bloom filters**: Clearing a bit to `0` could break other keys that hashed to the same bit (causing false negatives). Supporting deletes requires a **Counting Bloom Filter** (multi-bit counters instead of single bits, using 3–4× more memory) or a **Cuckoo Filter**.
> - **Saturation**: As the number of inserted elements exceeds the planned capacity $n$, the bit array fills with `1`s and the false-positive rate approaches 100%, silently degrading performance (e.g., turning [[LSM Tree]] point lookups into full disk reads).
> - **Rule of thumb**: ~10 bits per element gives roughly a **1% false-positive rate** (with $k \approx 7$ hash functions).

Bloom filter membership guarantee:::No false negatives (if it says "absent", the key is definitely absent), but possible false positives.

## Why can't you delete an element from a standard Bloom filter? #card
Multiple keys can hash to the same bit position. Flipping a bit back from `1` to `0` to delete one key could cause other inserted keys sharing that bit to falsely report as absent (introducing false negatives). Use a Counting Bloom Filter or Cuckoo Filter if deletions are required.

In [[LSM Tree]] engines, a Bloom filter per [[SSTable]] primarily optimizes ==point lookups for nonexistent keys== by skipping unnecessary disk reads.

## Related
- [[Bitmap]]
- [[LSM Tree]]
- [[SSTable]]
- [[Apache Cassandra]]
- [[Data Structures]]
