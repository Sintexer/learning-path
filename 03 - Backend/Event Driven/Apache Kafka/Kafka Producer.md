Producers write data to [[Kafka Topic|topics]]. Producers know which [[Kafka Partition|partition]] to write to (and which [[Kafka]] broker is its leader), ahead of write.

In case of Kafka broker failures, Producers will automatically recover. End-to-end [[Delivery Semantics]] depend on both producer durability settings and [[Kafka Consumer]] commit/deduplication logic.

## Idempotent Producer

The core mechanism: **broker-side deduplication using a Producer ID + per-partition sequence numbers.**

When you set `enable.idempotence=true`, here's what happens:

1. **On startup**, the producer requests a **Producer ID (PID)** from the broker (via an `InitProducerId` request). This is an internal identifier, invisible to your application code, unique to this producer instance's session.
2. **For every partition the producer writes to**, it maintains a **monotonically increasing sequence number**, starting at 0, specific to that `(PID, partition)` pair. Every batch sent includes this sequence number alongside the actual message data.
3. **The broker tracks, per partition, the last sequence number it has committed for each PID.** When a new produce request arrives, the broker checks the incoming sequence number against what it last saw:
    - If it's **exactly last-seen + 1** → normal case, append it, advance the counter.
    - If it's a **duplicate** (sequence number ≤ what's already committed) — meaning the producer's original request actually succeeded, but the _acknowledgment_ got lost (e.g., network blip), so the client's retry logic resent it — the broker **recognizes the duplicate and does not append it again.** It just responds "success" as if it had, since it already has that data. **This is the actual deduplication step** — it happens broker-side, transparently.
    - If there's a **gap** (sequence number higher than expected) — that signals something went wrong (message loss or reordering), and the broker rejects it with an `OutOfOrderSequenceException`, surfacing a real problem rather than silently corrupting order.
4. This requires `acks=all` (the leader must confirm the write is fully replicated before acking) — otherwise the whole guarantee is meaningless, since you'd be deduping against a write that could still be lost.

**One critical scope limitation, worth stating explicitly if asked (it's the kind of nuance that separates surface knowledge from real understanding):** plain idempotence (`enable.idempotence=true` without a `transactional.id`) only prevents duplicates **from retries within a single producer session.** If the producer process itself **crashes and restarts**, it gets a brand-new PID on reconnect — it has no memory of the old session's sequence numbers, so it **cannot** deduplicate against writes from before the crash. That cross-session guarantee is what Kafka's full **transactional producer** (`transactional.id` set, enabling actual transactions / [[Exactly-once delivery]]) adds on top — it lets a producer resume its identity across restarts (fencing zombie producers via epochs). Plain idempotence solves "retry-on-timeout duplicates," not "crash-and-restart duplicates" — those need transactions or your own [[Idempotency Key]] / [[Version Guard]] logic downstream.

> [!warning] Failure modes & trade-offs
> - **`acks=all` is NOT durable alone:** If `min.insync.replicas=1` (the default) and all followers lag behind, the leader is the only ISR—`acks=all` succeeds on a single broker and data is lost if that broker crashes. Pair `acks=all` with `replication.factor >= 3` and `min.insync.replicas >= 2`.
> - **Ordering under retries:** With `enable.idempotence=true` (default in Kafka 3.0+), `max.in.flight.requests.per.connection` must be `<= 5` to preserve per-partition ordering while pipelining batches.
> - **Producer buffer [[Backpressure]]:** If brokers slow down and the producer's `buffer.memory` fills up, `send()` blocks up to `max.block.ms` and then throws `TimeoutException`—never assume `send()` is infinitely non-blocking.

## Flashcards

## What is the scope limitation of `enable.idempotence=true` without a `transactional.id`? #card
Broker-side `(PID, sequenceNumber)` deduplication only prevents duplicates caused by **network retries within a single producer session**. If the producer crashes and restarts, it receives a new PID and cannot deduplicate records re-sent from before the crash—solving that requires `transactional.id` or downstream [[Idempotent Consumer]] / [[Idempotency Key]] checks.

Why must producer `acks=all` be paired with broker `min.insync.replicas >= 2` (and `replication.factor >= 3`)?::Without `min.insync.replicas >= 2`, if all followers fall out of sync, the lone leader satisfies `acks=all` by itself and loses committed messages if it crashes before replicating.

How does a Kafka broker detect and deduplicate retried batches from an idempotent producer?::It tracks the last committed monotonic ==sequence number per `(Producer ID, partition)`==; if an incoming batch has a sequence number ≤ last committed, the broker acks success without appending a duplicate.

## Related
- [[Kafka]]
- [[Kafka Topic]]
- [[Kafka Partition]]
- [[Kafka Message]]
- [[Kafka Consumer]]
- [[Delivery Semantics]]
- [[At-least-once delivery]]
- [[Exactly-once delivery]]
- [[Idempotency Key]]
- [[Outbox Pattern]]
- [[Dual-Write]]