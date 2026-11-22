[[Kafka Topic]]s are split into ***partitions***. Messages within each partition are strictly ordered. When no key is provided, modern Kafka producers use a sticky partitioner (filling a batch for one partition before switching) rather than per-message round-robin. To determine the partition based on the key, [[Kafka]] applies a hashing algorithm (`murmur2`):

```java
var targetPartition = Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
```

Data in topics is kept only for a limited time (default is one week; see [[Kafka Retention]]).

Each message within a partition gets an incremental id, called ***offset***. Offsets only have a meaning for a specific partition. Offsets are not re-used even if previous messages have been deleted.

Kafka Topics are *immutable*: once data is written to a partition, it cannot be changed.

## Why partition at all?

Because a single log = single point of throughput. If everything went into one giant ordered log, only one [[Kafka Consumer|consumer]] process could realistically read it fast enough at scale. By splitting into partitions, Kafka lets you **parallelize** — multiple consumers in a consumer group can each read a different partition simultaneously (and each partition acts like a [[Single Writer Principle]] lane for its assigned consumer).

But this creates a problem: if you split messages across partitions randomly, you lose ordering for related messages. That's solved by the partition key.

### The Partition Key

When a [[Kafka Producer|producer]] sends a message, it can attach a **key** (commonly something like `orderId`, `userId`, `betId`). Kafka runs that key through a hash function to decide _which partition_ the message goes to.

The guarantee this gives you: **every message with the same key always lands in the same partition**, and therefore is always delivered in the order it was produced, relative to other messages with that same key.

This is the detail most people gloss over but that senior engineers lean on constantly: **"ordering is guaranteed per-key, via partitioning"** — not per-topic.

> [!warning] Failure modes & trade-offs
> - **Increasing partition count breaks per-key ordering:** Because Kafka routes keys via `hash(key) % numPartitions` (not [[Consistent Hashing]]), adding partitions remaps existing keys to new partitions. While old messages drain from the old partition and new ones land on the new partition, consumers can process events for the same key **out of order**. Plan partition counts ahead of time.
> - **Hot partitions (key skew):** If one key (e.g. a mega-merchant or viral event) generates 40% of traffic, its single partition saturates one consumer while other partitions sit idle—choose a high-cardinality key or salt hot keys when strict global ordering per hot key is not required.
> - **Partition count caps parallelism:** Within one consumer group, you can have at most **1 active consumer per partition**; extra consumers sit idle. Conversely, excessive partitions increase broker memory, open file handles, and failover time.

## Flashcards

In a single Kafka consumer group, the maximum number of active parallel consumers for a topic is bounded by the topic's ==number of partitions==.

## Why is increasing a Kafka topic's partition count risky for keyed streams? #card
Kafka maps keys using `murmur2(key) % numPartitions`. Changing `numPartitions` remaps existing keys to different partitions, so new events for key `K` may be processed on the new partition before older events for `K` finish draining from the old partition—violating per-key ordering.

Choose partition keys with high cardinality to prevent hot partitions (key skew) where a single consumer becomes a bottleneck while other partitions sit idle. #card-reminder

## Related
- [[Kafka]]
- [[Kafka Topic]]
- [[Kafka Producer]]
- [[Kafka Consumer]]
- [[Kafka Retention]]
- [[Data partitioning]]
- [[Single Writer Principle]]
- [[Consistent Hashing]]
