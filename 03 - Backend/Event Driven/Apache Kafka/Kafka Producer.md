Producers write data to topics. Producers know to which partition to write to (and which [[Kafka]] broker has it), ahead of write.

In case of Kafka broker failures, Producers will automatically recover. [[Delivery Semantics]] is controlled by [[Kafka Consumer]]'s code.

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

**One critical scope limitation, worth stating explicitly if asked (it's the kind of nuance that separates surface knowledge from real understanding):** plain idempotence (`enable.idempotence=true` without a `transactional.id`) only prevents duplicates **from retries within a single producer session.** If the producer process itself **crashes and restarts**, it gets a brand-new PID on reconnect — it has no memory of the old session's sequence numbers, so it **cannot** deduplicate against writes from before the crash. That cross-session guarantee is what Kafka's full **transactional producer** (`transactional.id` set, enabling actual transactions) adds on top — it lets a producer resume its identity across restarts. Plain idempotence solves "retry-on-timeout duplicates," not "crash-and-restart duplicates" — those need transactions or your own idempotency-key/version-guard logic downstream.