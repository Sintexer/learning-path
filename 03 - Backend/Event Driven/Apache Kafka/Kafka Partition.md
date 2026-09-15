[[Kafka Topic]]s are split in ***partitions***. Messages within each partition are ordered. Data is assigned by a round-robin algorithm to a random partition unless a key is provided. To determine the partition based on the key, [[Kafka]] applies a hashing algorithm (`murmur2`):

```java
var targetPartition = Math.abs(Utils.murmur2(keyBytes)) % (numPartitions - 1)
```

Data in topics is kept only for a limited time (default is one week)

Each message within a partition gets an incremental id, called ***offset***. Offset only have a meaning for a specific partition. Offset are not re-used even if previous messages have been deleted.

Kafka Topics are *immutable*: once data is written to a partition, it cannot be changed.

## Why partition at all?

Because a single log = single point of throughput. If everything went into one giant ordered log, only one consumer process could realistically read it fast enough at scale. By splitting into partitions, Kafka lets you **parallelize** — multiple consumers can each read a different partition simultaneously.

But this creates a problem: if you split messages across partitions randomly, you lose ordering for related messages. That's solved by the partition key.

### The Partition Key

When a producer sends a message, it can attach a **key** (commonly something like `orderId`, `userId`, `betId`). Kafka runs that key through a hash function to decide _which partition_ the message goes to.

The guarantee this gives you: **every message with the same key always lands in the same partition**, and therefore is always delivered in the order it was produced, relative to other messages with that same key.

This is the detail most people gloss over but that senior engineers lean on constantly: **"ordering is guaranteed per-key, via partitioning"** — not per-topic.
