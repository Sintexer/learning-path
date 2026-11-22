---
flashcards:
  q-fyf8: { nid: 1795356733268, hash: r9t75sr2, sync: ncxy9ivg }
  q-ppv8: { nid: 1795356733298, hash: fz9xtpcb, sync: 38sd5nsa }
  q-39kr: { nid: 1795356733319, hash: vhnmm4se, sync: 6f32edqb }
---
A **Kafka message** (also called a **record**) is the atomic unit of data written by a [[Kafka Producer]] and stored sequentially inside a [[Kafka Partition]]. Brokers treat keys and values as opaque byte arrays—schema and [[Data Encoding]] are managed by clients (JSON, Avro, [[Protocol Buffers]]).

## Key 

`binary`, optional (can be `null`).
- Determines target [[Kafka Partition]] via `murmur2(key) % numPartitions` to guarantee per-key ordering.
- Required for compacted topics (see [[Kafka Topic]]).

## Value

`binary`, optional (can be `null`).
- Holds the serialized payload.
- A `null` value with a non-null key acts as a **tombstone** (delete marker) in compacted topics.

## Compression type

Configured on the producer (`compression.type`) and applied at the **record-batch level** (not per individual message), so larger batches (`batch.size`, `linger.ms`) yield much better compression ratios:
- `none`
- `gzip` (higher compression ratio, higher CPU)
- `snappy` (fast, moderate ratio)
- `lz4` (very fast, great default for low-latency throughput)
- `zstd` (best ratio-to-CPU balance on modern Kafka)

## Headers

Optional list of key-value metadata pairs (UTF-8 key, `byte[]` value).
- Ideal for routing/observability metadata without deserializing the payload: [[Distributed Tracing]] / [[OpenTelemetry]] `traceparent`, event type, schema version, or [[Idempotency Key]].

## Partition 

Message's target [[Kafka Partition]] chosen by the [[Kafka Producer]]'s partitioner.

## Offset

Monotonically increasing 64-bit position assigned by the partition leader broker upon append; unique only within that partition.

## Timestamp

64-bit epoch millisecond timestamp—either `CreateTime` (set by producer, default) or `LogAppendTime` (overwritten by broker on append). Used for time-based [[Kafka Retention]] and timestamp-based offset lookups.

> [!warning] Failure modes & trade-offs
> - **Oversized messages (> 1 MB default `message.max.bytes`):** Pushing multi-megabyte blobs through Kafka degrades broker page-cache efficiency, spikes GC/replication latency, and stalls consumers. Use the **Claim Check pattern** (store the large payload in S3/object storage and send only its URI + metadata in the Kafka message).
> - **Compression mismatch:** Setting topic-level compression differently from producer compression forces brokers to decompress and recompress every batch—prefer `compression.type=producer` on brokers so brokers keep producer-compressed batches intact.

## Flashcards

In Kafka, compression (`lz4`, `zstd`, `snappy`, `gzip`) is applied at the ==record-batch== level rather than per individual message, so producer batching (`linger.ms` / `batch.size`) directly improves compression ratio.
^q-fyf8

What is the Claim Check pattern for large payloads in [[Kafka]]?::Store the large binary payload in external object storage (e.g. S3) and publish a small [[Kafka Message]] containing only the storage reference/URI and metadata, keeping broker throughput and page cache healthy.
^q-ppv8

Put cross-cutting metadata (like [[OpenTelemetry]] trace context or event type) in Kafka message headers so consumers and routers can inspect it without deserializing the payload. #card-reminder
^q-39kr

## Related
- [[Kafka]]
- [[Kafka Topic]]
- [[Kafka Partition]]
- [[Kafka Producer]]
- [[Kafka Retention]]
- [[Data Encoding]]
- [[Distributed Tracing]]
- [[OpenTelemetry]]