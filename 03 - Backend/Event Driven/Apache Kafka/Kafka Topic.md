---
flashcards:
  q-qbcr: { nid: 1795356732921, hash: uesm84dj, sync: fnpgnqdy }
  q-dtky: { nid: 1795356732941, hash: 3dqsm8uj, sync: cjkzca62 }
  q-657w: { nid: 1795356732966, hash: fa5rjznv, sync: gz7arjkv }
---

Topic is a *named* stream of data. It is like a table in the database. You can have as many topics as you need. Topic supports any kind of message format.

The sequence of messages in a topic is called a **data stream**.

You cannot query topics. Use [[Kafka Producer]] to send data and [[Kafka Consumer|Kafka Consumers]] to read the data.

## Compacted Topic

By default, [[Kafka]] retains messages based on **time or size** (`retention.ms`, `retention.bytes`; see [[Kafka Retention]]) — old messages get deleted once they age out or the topic exceeds a size threshold, regardless of key.

**Log compaction** is a different retention strategy: instead of deleting by age, Kafka guarantees it will retain **at least the most recent message for every unique key**, and is free to discard older messages for that same key. Effectively, a compacted topic behaves like a **durable key-value store** — "what's the latest value for key X" — rather than a full history log.

Configured via `cleanup.policy=compact` on the topic (as opposed to the default `delete`).

**Where this is genuinely useful:**

- A "current state" cache (e.g., "latest odds per match," "latest known status per order") where you only ever care about the newest value, not history
- Internally, Kafka itself uses a compacted topic for `__consumer_offsets` — it only needs the _latest_ committed offset per `(group, partition)`, not the full history of every commit ever made
- **Explicitly not appropriate for [[Event Sourcing|event sourcing]]** — compaction destroys the history that event sourcing depends on

One nuance worth knowing: compaction doesn't happen instantly — it runs periodically in the background, so a compacted topic can briefly contain multiple values for the same key before cleanup catches up. Consumers should be written expecting that, not assuming perfect single-value-per-key at all times.

> [!warning] Failure modes & trade-offs
> - **Deleting keys requires tombstones:** To remove a key from a compacted topic, produce a [[Kafka Message]] with that key and a `null` value (a tombstone). Brokers retain tombstones for `delete.retention.ms` (default 24h) so consumers can observe the deletion before the key disappears completely.
> - **Active segment is never compacted:** The currently open log segment is excluded from compaction until it rolls over, so newly written duplicate keys remain visible until segment roll + cleaner pass.
> - **Compacted topics require non-null keys:** Producing a message with a `null` key to a compacted topic fails because compaction groups records by key.

## Flashcards

How does a compacted Kafka topic (`cleanup.policy=compact`) differ from the default `delete` policy?::Instead of deleting records by age or size, Kafka retains at least the ==latest value for each unique key== (behaving like a durable key-value changelog rather than a full event history).
^q-dtky
^q-qbcr

## How do you delete a key from a compacted Kafka topic, and why does `delete.retention.ms` matter? #card
Publish a **tombstone** record—a message with the target key and a `null` value. Brokers keep the tombstone for `delete.retention.ms` so downstream consumers have time to observe the delete before the log cleaner removes the key permanently.

Why is a compacted Kafka topic unsuitable for [[Event Sourcing]]?::Log compaction discards older events for the same key, destroying the historical sequence of state transitions that [[Event Sourcing]] requires to rebuild state.
^q-657w

## Related
- [[Kafka]]
- [[Kafka Partition]]
- [[Kafka Retention]]
- [[Kafka Message]]
- [[Event Sourcing]]
- [[Change Data Capture]]