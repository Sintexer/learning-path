---
flashcards:
  q-q4yx: { nid: 1795356721417, hash: xc8rt854, sync: b6dp33hn }
  q-3ca9: { nid: 1795356721442, hash: 228a8khm, sync: z9pw8r7m }
  q-taww: { nid: 1795356721467, hash: 84y3qqdn, sync: kxn34mnv }
---

**Monotonic version-stamp style** (a flavor of [[Optimistic Concurrency Control]]) — "Apply this change if its version is _strictly greater than_ what I currently have." More lenient than strict [[Compare-And-Swap]] — tolerates gaps (e.g., missing sequence 46 doesn't block 47 or 48 from applying), which is usually what you want for **read models ([[CQRS]] projections) built from event streams**, since perfect gapless delivery isn't always realistic (compaction, selective replay, [[DLQ]] redelivery, etc.). This is the version most relevant to [[Kafka Consumer]] replay-safety and [[Idempotent Consumer]] design.

Highly related to [[Kafka Consumer]].

### Why not use wall-clock timestamps as the version?

Suppose you use a plain **timestamp** as your version field instead of a monotonically incrementing integer. What's a real risk with that choice, specifically in a distributed system with multiple producers?

> Timestamps might overlap. But there is a bigger problem: **clock skew**. If Producer A and Producer B run on different servers, their system clocks are never perfectly synchronized (NTP sync error is normal, usually milliseconds, sometimes more, and NTP can jump backward). So imagine Producer A generates event X at "true" time T, and Producer B generates event Y at "true" time T+1 (genuinely, causally later). But because Producer B's clock is running 5ms behind, event Y gets timestamped as _earlier_ than event X. Your version guard now thinks Y is stale and incorrectly discards a legitimately newer update. **A monotonically incrementing counter, maintained by a single authoritative source (usually the write-side database, see [[Single Writer Principle]]), doesn't have this problem** — it has no relationship to wall-clock time at all, so there's no skew to worry about. This is why real systems almost always prefer sequence numbers / logical versions (or [[Fencing Token|fencing tokens]]) over timestamps for this exact purpose.

> [!tip] Production reality
> Implement the version guard directly in SQL as an [[Atomic Conditional Mutation]]:
> `UPDATE order_view SET status = :status, version = :ver WHERE id = :id AND version < :ver;`
> This eliminates [[Check-Then-Act]] races and makes out-of-order replays and duplicate deliveries completely harmless in a single statement.

Version Guard:::A monotonic-version check (`incoming_version > stored_version`) that ignores stale or duplicate updates while tolerating sequence gaps — ideal for replay-safe [[CQRS]] projections and [[Kafka Consumer|Kafka consumers]].
^q-q4yx

## Why should a Version Guard use a monotonically incrementing counter from a single writer instead of wall-clock timestamps? #card
Wall-clock timestamps suffer from **NTP clock skew** and backward clock jumps across servers: a causally newer event from a server whose clock lags behind gets a smaller timestamp and is **silently discarded as stale**. A monotonic integer counter owned by a single authoritative writer avoids clock skew entirely.
^q-3ca9

Enforce a Version Guard directly inside the SQL `WHERE` clause (`WHERE id = :id AND version < :ver`) so the check and update are ==atomic== and immune to ==Check-Then-Act== races. #card-reminder
^q-taww

## Related
- [[Optimistic Concurrency Control]]
- [[Atomic Conditional Mutation]]
- [[Idempotent Consumer]]
- [[Kafka Consumer]]
- [[CQRS]]
- [[DLQ]]
- [[Single Writer Principle]]
- [[Fencing Token]]
- [[Check-Then-Act]]
