---
flashcards:
  q-g7uv: { nid: 1795356724341, hash: btij3j6i, sync: kiur997e }
  q-wmpw: { nid: 1795356724366, hash: nfzqq5sc, sync: n6jutfut }
---

Messages are delivered zero or one times — never duplicated, but may be lost on failure.

Commit the offset **before** processing the message (or send fire-and-forget without producer retries).

- If the consumer crashes mid-processing, the offset is already committed → on restart, [[Kafka]] thinks that message is done → it's **never redelivered** → the work is simply lost.
- Rarely intentional. You'd only choose this if losing occasional messages is truly acceptable and reprocessing/duplicates would be worse (rare in practice — e.g., high-volume telemetry, non-critical metrics, or best-effort logging).

> [!tip] Production reality
> Teams rarely choose at-most-once deliberately, yet hit it accidentally via `enable.auto.commit=true` in [[Kafka Consumer|Kafka consumers]] or fire-and-forget HTTP/UDP calls without retries. Always verify when offsets are committed relative to durable state writes.

At-most-once delivery commits the consumer offset ==before== processing finishes, preventing duplicates at the cost of ==possible message loss== on crash.
^q-g7uv

When is at-most-once delivery an intentional design choice?::High-volume, non-critical streams (metrics, click telemetry, best-effort logs) where occasional loss is acceptable and retry/deduplication overhead is not worth it.
^q-wmpw

## Related
- [[Delivery Semantics]]
- [[At-least-once delivery]]
- [[Exactly-once delivery]]
- [[Kafka Consumer]]
- [[UDP]]
