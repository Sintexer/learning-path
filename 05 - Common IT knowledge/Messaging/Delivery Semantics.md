---
flashcards:
  q-athr: { nid: 1795356724292, hash: ive4v8h2, sync: 65mpnwg3 }
  q-ybpx: { nid: 1795356724317, hash: 5mznh94t, sync: xce3y6cc }
---

Delivery semantics define the contract between a message producer, broker (e.g., [[Kafka]]), and consumer regarding **message loss** vs **message duplication** when networks drop packets or processes crash.

- [[At-most-once-delivery]] — zero or one delivery; fast, no duplicates, but messages can be lost on failure.
- [[At-least-once delivery]] — one or more deliveries; no silent loss, but consumers must tolerate duplicates.
- [[Exactly-once delivery]] — each message's effect is applied once; in practice achieved via at-least-once transport plus idempotent side effects (or broker-scoped transactions).

## Comparison

| Semantic | Offset Commit Timing | Message Loss? | Duplicates? | Typical Use Case |
| --- | --- | --- | --- | --- |
| **[[At-most-once-delivery\|At-most-once]]** | Before processing | Yes | No | Non-critical metrics, best-effort logs |
| **[[At-least-once delivery\|At-least-once]]** | After processing | No | Yes | Default for almost all business workflows |
| **[[Exactly-once delivery\|Exactly-once]]** | Atomic with state write (or deduplicated) | No | Effect applied once | Payments, ledgers, Kafka-to-Kafka stream transforms |

## End-to-End Scope Matters

Delivery semantics are **end-to-end**, not just a broker toggle. Every hop can weaken the guarantee:

1. **Producer → Broker**: Needs durable acks (`acks=all` + idempotent producer) and [[Outbox Pattern]] / [[Change Data Capture]] to avoid [[Dual-Write]] loss.
2. **Broker → Consumer**: Determined by when offsets are committed relative to side-effect persistence.
3. **Consumer → Downstream DB / API**: Requires [[Idempotent Consumer]], [[Idempotency Key]], or [[Version Guard]] so retries do not corrupt state.

> [!warning] Failure modes & trade-offs
> Turning on "exactly-once" inside a broker (e.g., Kafka transactions) only covers **consume → transform → produce** within the same Kafka cluster. The moment a consumer writes to an external database or calls an external HTTP API, you are back to [[At-least-once delivery]] and must enforce [[Idempotency]] at the boundary.

## Why is "exactly-once" an end-to-end property rather than a broker setting? #card
A broker can only coordinate its own topics and offsets. As soon as a consumer updates an external database or calls a third-party API, a crash between the external side effect and the offset commit causes redelivery — requiring [[Idempotency]] (e.g., [[Idempotency Key]] or deduplication table) at the sink.
^q-athr

Design every distributed consumer for ==at-least-once delivery== and achieve an effectively ==exactly-once effect== via idempotent processing. #card-reminder
^q-ybpx

## Related
- [[At-most-once-delivery]]
- [[At-least-once delivery]]
- [[Exactly-once delivery]]
- [[Idempotent Consumer]]
- [[Outbox Pattern]]
- [[Dual-Write]]
- [[Kafka]]
