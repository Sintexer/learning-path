---
aliases:
  - OCC
flashcards:
  q-di6m: { nid: 1795356721194, hash: q54fsu8k, sync: 3fduxsdr }
  q-htyi: { nid: 1795356721215, hash: fvebmmuk, sync: 9q2isnkj }
---
**Definition:** Optimistic Concurrency Control (OCC) is a strategy for handling concurrent updates _without acquiring locks upfront_. It assumes conflicts are rare, lets operations proceed freely, and instead checks — at the moment of writing — whether the data is still in the state you expect. If not, you reject, skip, or retry the operation rather than corrupt state.

In-memory and low-level optimistic strategies take a copy of the data, modify it, and write the changes back only if the data has not mutated in the meantime. If a concurrent change happened, the operation repeats until it succeeds. Under high contention, repeated retries create a queuing effect similar to [[Mutual Exclusion]]. Optimistic strategies work well for data, but not for non-copyable hardware resources. Hardware-level atomicity is provided by [[Compare-And-Swap|CAS]] instructions. Notably, most locking strategies use optimistic primitives underneath to change the lock state — **to build a "Pessimistic" lock, you use an "Optimistic" technique underneath.**

**Contrast with pessimistic locking:** [[Pessimistic Locking]] (e.g., SQL `SELECT ... FOR UPDATE`) acquires a lock _before_ you even read the data, blocking any other [[Transaction|transaction]] from touching it until you're done. Safe, but expensive — reduces concurrency, holds resources, and can cause deadlocks under load. OCC is the "ask forgiveness, not permission" model, better suited for high-throughput systems like event streams where locking every consumer against every other would defeat the purpose entirely. See also [[Optimistic Locking]] for database-level row versioning and [[Pessimistic Concurrency Control]].

**Two flavors of OCC you should be able to distinguish:**

1. **[[Compare-And-Swap]] / expected-version style** — "Apply this change _only if_ the current version exactly equals X (the version I expected when I computed this change)." This is strict — any mismatch, even from a gap, causes rejection. Common in systems needing exact causal chains (e.g., "apply this diff only if nothing else changed since I last read").
2. **Monotonic version-stamp style** ([[Version Guard]]) — "Apply this change if its version is _strictly greater than_ what I currently have." More lenient — tolerates gaps (e.g., missing sequence 46 doesn't block 47 or 48 from applying), which is usually what you want for **read models built from event streams**, since perfect gapless delivery isn't always realistic (compaction, selective replay, etc.). This is the version most relevant to [[Kafka Consumer]] replay-safety.

**Where does the version number come from?** It must be assigned by a **single authoritative source** per entity — usually whichever service owns the write-side truth for that entity (the feed provider owning match odds, or your core Bets service owning bet state — see [[Single Writer Principle]]). Never let multiple independent producers assign versions to the same entity's version counter — that reintroduces exactly the race-condition problem we're trying to avoid.

> [!warning] Failure modes & trade-offs
> Under **hot-key contention** (many writers hammering the same row, e.g., a flash-sale inventory counter), OCC degrades into **livelock / retry storms**: 90%+ of transactions do work, fail at commit, and retry, burning CPU and DB I/O. For hot keys, switch to [[Pessimistic Locking]], [[Atomic Conditional Mutation]] (`UPDATE ... SET stock = stock - 1 WHERE stock > 0`), or shard the counter.

## What is the difference between strict CAS-style OCC and monotonic Version Guard OCC? #card
- **Strict [[Compare-And-Swap|CAS]] (`version = expected`)**: Rejects any mismatch including gaps; used for write-side state transitions where exact causal history matters.
- **Monotonic [[Version Guard]] (`incoming_version > stored_version`)**: Tolerates sequence gaps (e.g., Kafka compaction/replay); ideal for read-model projections where latest state wins.

When does Optimistic Concurrency Control perform worse than Pessimistic Locking?::Under high contention on hot keys, where repeated conflict-and-retry loops waste CPU/DB I/O and cause retry storms or starvation.
^q-di6m

Version numbers for OCC must come from a ==single authoritative writer== per entity; multiple independent producers assigning versions reintroduce race conditions. #card-reminder
^q-htyi

## Related
- [[Optimistic Locking]]
- [[Pessimistic Locking]]
- [[Pessimistic Concurrency Control]]
- [[Compare-And-Swap]]
- [[Version Guard]]
- [[Atomic Conditional Mutation]]
- [[Check-Then-Act]]
- [[Single Writer Principle]]
- [[Fencing Token]]
