It is guaranteed that messages will be delivered and there won't be delivery duplications.

The message is processed **and its effect applied exactly one time**, no duplicates, no loss — semantically.

**True "exactly-once" transport, in the general distributed systems sense, is essentially unachievable** — you can't have a network call plus a state change be perfectly atomic across two independent systems without prohibitive coordination cost (the Two Generals Problem).

What real systems do instead is:

**[[At-least-once delivery]] + [[Idempotent Consumer|idempotent processing]] = "exactly-once effect."**

In other words: let [[Kafka]] redeliver duplicates freely (that's fine, that's just [[At-least-once delivery]]), but design your consumer logic (using [[Version Guard|version guards]], [[Idempotency Key|unique operation IDs]], or naturally idempotent operations) so that processing the same message twice produces the **same end state** as processing it once. The _transport_ is at-least-once; the _outcome_ behaves like exactly-once. This is the practical, industry-standard way almost every "exactly-once" system claim actually works under the hood.

> [!tip] Production reality
> [[Kafka]]'s built-in EOS (`enable.idempotence=true` + transactional producer + `read_committed` consumers) gives genuine exactly-once semantics **only for read-process-write pipelines staying entirely inside Kafka**. For external sinks (like [[PostgreSQL]]), store the consumer's processed message ID or offset **in the same local DB transaction** as the business mutation, or use an [[Idempotency Key]] with a unique constraint.

In distributed systems, practical "exactly-once" is achieved by combining ==at-least-once delivery== with ==idempotent processing==.

## What is the scope limitation of Kafka's native Exactly-Once Semantics (EOS)? #card
Kafka EOS only guarantees atomic offset commit + output publish within **Kafka-to-Kafka** pipelines. Once a consumer writes to an external database or calls an external API, you must pair [[At-least-once delivery]] with transactional deduplication (storing offset/event ID in the same DB transaction) or an [[Idempotency Key]].

## Related
- [[Delivery Semantics]]
- [[At-least-once delivery]]
- [[At-most-once-delivery]]
- [[Idempotent Consumer]]
- [[Idempotency Key]]
- [[Version Guard]]
- [[API Idempotency]]
- [[Kafka]]
