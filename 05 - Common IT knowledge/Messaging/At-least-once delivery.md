A message might be delivered and processed more than once in case of retries or consumer restarts.

Commit the offset **after** successful processing (see [[Kafka Consumer]]).

- If the consumer crashes _after_ finishing the work but _before_ the commit lands, on restart it re-reads from the last committed offset — meaning it **reprocesses** that message again.
- Consequence: **duplicates are possible**, but **nothing is ever silently lost**.
- This is the **default, practical choice** in almost all real systems — because losing data is almost always worse than occasionally doing something twice, _provided_ you've made the processing safe to repeat (see [[Idempotent Consumer]]).

> [!warning] Failure modes & trade-offs
> At-least-once delivery only holds if the producer also retries until broker acknowledgment (`acks=all` in [[Kafka]]) and the consumer commits offsets **strictly after** side effects are durably persisted. If a consumer commits asynchronously in the background (`enable.auto.commit=true`), a crash mid-batch silently degrades your guarantee to [[At-most-once-delivery]].

At-least-once delivery commits the consumer offset ==after== processing finishes, preventing message loss at the cost of ==possible duplicates==.

## How does at-least-once delivery achieve an effectively exactly-once outcome? #card
Pair at-least-once transport (producer retries + post-processing offset commit) with an [[Idempotent Consumer]] using an [[Idempotency Key]], [[Version Guard]], or atomic deduplication table so redelivered messages produce the same final state.

## Related
- [[Delivery Semantics]]
- [[At-most-once-delivery]]
- [[Exactly-once delivery]]
- [[Idempotent Consumer]]
- [[Idempotency Key]]
- [[Version Guard]]
- [[Kafka Consumer]]
