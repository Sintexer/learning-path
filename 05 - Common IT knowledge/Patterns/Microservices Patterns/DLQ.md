**Dead Letter Queue (DLQ)**

The problem: what happens when a [[Kafka Consumer|consumer]] tries to process a message and it **keeps failing** — a bug in the payload, an unexpected null field, a schema mismatch, or a downstream dependency that's permanently broken for this specific message (a "**poison pill**" message)?

If you just retry forever ([[Retry Pattern]]), that message **blocks all progress** on its [[Kafka Partition|partition]] — since [[Kafka]] delivers messages in order per partition, nothing after it can be processed until it succeeds (or you skip it).

The standard solution: after a bounded number of retry attempts, instead of retrying indefinitely, the consumer **publishes the problematic message to a separate topic** — the **Dead Letter Queue** (often literally named something like `orders-events-dlq`) — along with diagnostic metadata (original topic/partition/offset, exception stack trace, retry count, timestamp) and moves on, committing its offset past the poison pill. The DLQ topic becomes a place for engineers to **inspect, debug, and potentially reprocess** those failed messages later, without holding up the entire live pipeline.

> [!warning] Failure modes & trade-offs
> - **Ordering violation**: Moving message #10 to a DLQ lets message #11 for the same entity key process immediately. If you later replay #10 from the DLQ without a [[Version Guard]], stale #10 can overwrite newer state from #11!
> - **Write-only graveyard**: A DLQ without alerting, dashboards, retention policy, and a tested replay tool silently drops business events until customers complain.
> - **Transient vs permanent routing**: Routing transient downstream outages directly to the DLQ floods it with thousands of healthy messages. Distinguish **non-retryable poison pills** (deserialization/validation errors → immediate DLQ) from **transient errors** (backoff/retry topics or pause consumption with [[Backpressure]]).

Dead Letter Queue (DLQ):::A dedicated fallback topic/queue where a consumer routes unprocessable ("poison pill") messages after bounded retries so they do not block partition progress.

## What hidden data-corruption risk happens when you replay messages from a DLQ? #card
Routing message #10 to a DLQ allows newer messages (#11, #12) for the same entity key to process out of order. Replaying #10 later can **overwrite newer state** unless consumers enforce a [[Version Guard]] or [[Idempotent Consumer]] check.

Always pair a DLQ with ==alerting, failure metadata headers, and a safe replay mechanism== — otherwise it becomes a silent data graveyard. #card-reminder

## Related
- [[Kafka Consumer]]
- [[Kafka Partition]]
- [[Retry Pattern]]
- [[Version Guard]]
- [[Idempotent Consumer]]
- [[Reconciliation]]
- [[Backpressure]]
- [[Delivery Semantics]]
