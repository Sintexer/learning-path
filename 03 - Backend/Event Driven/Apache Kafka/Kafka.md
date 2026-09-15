
Kafka receives and outputs messages as bytes.

Because of that both [[Kafka Producer]] and [[Kafka Consumer]] must know Serialization and Deserialization details. The serialization/deserialization type must now change during a topic lifecycle (create a new topic instead).

## Kafka features

- Distributed
- Resilient
- Fault tolerant
- Horizontal scalability
- High performance
- Used by many high-end vendors, such as LinkedIn, Netflix, Uber, AirBnB
### The Physical Picture

Imagine Kafka not as a "message queue" (that's the wrong mental model), but as a **distributed, append-only log file system**.

- A **topic** is just a name/category — like "bet-events" or "user-signups."
- Under the hood, a topic is split into one or more **[[Kafka Partition|partitions]]**.
- Each partition is literally an **ordered log** — think of it like a file where new messages only get appended to the end. Never inserted in the middle, never reordered.
- Every message that lands in a partition gets a number — its **offset** — which is just "its position in that specific log." 0, 1, 2, 3...

**The critical consequence:** ordering is only guaranteed _within_ a single [[Kafka Partition|partitions]]. If topic "bet-events" has 3 partitions, and messages for bet #123 could theoretically end up split across partitions 0 and 2, there is **no guarantee** which one Kafka delivers "first" relative to the other. Ordering across partitions is undefined.