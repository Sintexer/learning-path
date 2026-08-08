Topic is a *named* stream of data. It is like a table in the database. You can have as many topics as you need. Topic supports any kind of message format.

The sequence of messages in a topic is called a **data stream**.

You cannot query topics. Use [[Kafka Producer]] to send data and **Kafka Consumers** to read the data.

Topics are split in ***partitions***. Messages within each partition are ordered. Data is assigned by a round-robin algorithm to a random partition unless a key is provided. To determine the partition based on the key, Kafka applies a hashing algorithm (`murmur2`):

```java
var targetPartition = Math.abs(Utils.murmur2(keyBytes)) % (numPartitions - 1)
```

Data in topics is kept only for a limited time (default is one week)

Each message within a partition gets an incremental id, called ***offset***. Offset only have a meaning for a specific partition. Offset are not re-used even if previous messages have been deleted.

Kafka Topics are *immutable*: once data is written to a partition, it cannot be changed.