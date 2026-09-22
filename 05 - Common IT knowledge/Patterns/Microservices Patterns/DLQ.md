Dead Letter Queue

The problem: what happens when a consumer tries to process a message and it **keeps failing** — a bug in the payload, an unexpected null field, a downstream dependency that's permanently broken for this specific message (a "**poison pill**" message)?

If you just retry forever, that message **blocks all progress** on its partition — since Kafka delivers messages in order per partition, nothing after it can be processed until it succeeds (or you skip it).

The standard solution: after a bounded number of retry attempts, instead of retrying indefinitely, the consumer **publishes the problematic message to a separate topic** — the **Dead Letter Queue** (often literally named something like `orders-events-dlq`) — and moves on, committing its offset past the poison pill. The DLQ topic becomes a place for engineers to **manually inspect, debug, and potentially reprocess** those failed messages later, without holding up the entire live pipeline.