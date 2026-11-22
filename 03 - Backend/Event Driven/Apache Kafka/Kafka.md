---
flashcards:
  q-mhiz: { nid: 1795356733017, hash: 6ijf43zy, sync: kuhdfsxm }
  q-frz3: { nid: 1795356733041, hash: j4h8v7gj, sync: jzjruzc3 }
  q-2nwm: { nid: 1795356733067, hash: ht6fzfxh, sync: xuv9iaya }
---
Kafka receives and outputs messages as bytes (see [[Kafka Message]] and [[Data Encoding]]).

Because of that both [[Kafka Producer]] and [[Kafka Consumer]] must know Serialization and Deserialization details. The serialization/deserialization type must not change during a [[Kafka Topic|topic]] lifecycle (create a new topic instead, or evolve schemas with [[Backward Compatibility]] / [[Forward Compatibility]] using [[Protocol Buffers]] or Avro + Schema Registry).

## Kafka features

- Distributed
- Resilient
- Fault tolerant ([[Leader-Based Replication]] across brokers)
- [[Horizontal scaling|Horizontal scalability]]
- High performance
- Used by many high-end vendors, such as LinkedIn, Netflix, Uber, AirBnB

### The Physical Picture

Imagine Kafka not as a "message queue" (that's the wrong mental model), but as a **distributed, append-only log file system** (similar in spirit to a distributed [[WAL]]).

- A **[[Kafka Topic|topic]]** is just a name/category — like "bet-events" or "user-signups."
- Under the hood, a topic is split into one or more **[[Kafka Partition|partitions]]**.
- Each partition is literally an **ordered log** — think of it like a file where new messages only get appended to the end. Never inserted in the middle, never reordered.
- Every message that lands in a partition gets a number — its **offset** — which is just "its position in that specific log." 0, 1, 2, 3...

**The critical consequence:** ordering is only guaranteed _within_ a single [[Kafka Partition|partition]]. If topic "bet-events" has 3 partitions, and messages for bet #123 could theoretically end up split across partitions 0 and 2, there is **no guarantee** which one Kafka delivers "first" relative to the other. Ordering across partitions is undefined.

> [!tip] Production reality
> - **Why Kafka is fast:** Sequential append-only disk I/O + OS page cache + Linux zero-copy (`sendfile`) lets brokers stream gigabytes/sec without copying bytes into JVM heap.
> - **KRaft mode (modern Kafka):** Replaces ZooKeeper with an internal [[Raft]]-based metadata quorum inside Kafka controllers, simplifying operations and scaling to far more partitions.
> - **Kafka + DB reliability:** Publishing to Kafka and writing to a database in the same request is a [[Dual-Write]] hazard—pair Kafka with the [[Outbox Pattern]] or [[Change Data Capture]] (Debezium).

## Flashcards

In Apache Kafka, message ordering is guaranteed only within a single ==[[Kafka Partition|partition]]==, not across an entire [[Kafka Topic|topic]].
^q-mhiz

## Why is Kafka's mental model closer to a distributed append-only log than a traditional message queue? #card
Messages are appended immutably to partitioned logs with monotonic offsets and retained by time/size ([[Kafka Retention]]), not deleted upon consumption. Multiple independent [[Kafka Consumer|consumer groups]] can read and replay the same stream at their own pace without interfering with each other.
^q-frz3

Never perform a direct DB write and a Kafka publish in the same request without the [[Outbox Pattern]] or [[Change Data Capture]]—otherwise partial failures create [[Dual-Write]] inconsistency. #card-reminder
^q-2nwm

## Related
- [[Kafka Topic]]
- [[Kafka Partition]]
- [[Kafka Message]]
- [[Kafka Producer]]
- [[Kafka Consumer]]
- [[Kafka Retention]]
- [[Event-driven architecture]]
- [[Outbox Pattern]]
- [[Change Data Capture]]
- [[Delivery Semantics]]