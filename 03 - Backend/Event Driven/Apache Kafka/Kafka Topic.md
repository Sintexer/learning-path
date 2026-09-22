Topic is a *named* stream of data. It is like a table in the database. You can have as many topics as you need. Topic supports any kind of message format.

The sequence of messages in a topic is called a **data stream**.

You cannot query topics. Use [[Kafka Producer]] to send data and **Kafka Consumers** to read the data.

## Compacted Topic

By default, Kafka retains messages based on **time or size** (`retention.ms`, `retention.bytes`) — old messages get deleted once they age out or the topic exceeds a size threshold, regardless of key.

**Log compaction** is a different retention strategy: instead of deleting by age, Kafka guarantees it will retain **at least the most recent message for every unique key**, and is free to discard older messages for that same key. Effectively, a compacted topic behaves like a **durable key-value store** — "what's the latest value for key X" — rather than a full history log.

Configured via `cleanup.policy=compact` on the topic (as opposed to the default `delete`).

**Where this is genuinely useful:**

- A "current state" cache (e.g., "latest odds per match," "latest known status per order") where you only ever care about the newest value, not history
- Internally, Kafka itself uses a compacted topic for `__consumer_offsets` — it only needs the _latest_ committed offset per `(group, partition)`, not the full history of every commit ever made
- **Explicitly not appropriate for event sourcing** (as you correctly identified last topic) — compaction destroys the history that event sourcing depends on

One nuance worth knowing: compaction doesn't happen instantly — it runs periodically in the background, so a compacted topic can briefly contain multiple values for the same key before cleanup catches up. Consumers should be written expecting that, not assuming perfect single-value-per-key at all times.