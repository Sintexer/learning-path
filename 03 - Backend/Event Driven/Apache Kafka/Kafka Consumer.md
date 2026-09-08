Consumer receives the data from the broker's topic (identified by name) using the *pull model*. Consumers automatically know which broker to read from. In case of broker failures, consumers know how to recover. 

Data is read from low to high offset **within each partition**.

Kafka stores the offsets at which a consumer group has been reading. The offsets committed are in Kafka *topic* names `__consumer_offsets`. When a consumer has processed data received from Kafka, it should be periodically committing the offsets (the Kafka broker will write to `__consumer_offsets`, not the consumer group itself).

## Consumer groups

All the consumers in an application read data as a *consumer group*. Each consumer within a group reads from exclusive partitions. Each partition is read by one consumer in a consumer group. If there are more consumers than partitions, they should be split into more consumer groups.

The consumer group is controlled by the property `group.id`.


## Delivery semantics

On reconnect, consumer will start reading from the last committed offset. So there are several option to choose the delivery semantics based on whether the consumer tolerant for repeated messages or not:
1. **At least once** - (usually preferred) offsets are committed after the message is processed. If the processing goes wrong, the message will be read again. This can result in duplicate processing of messages.
2. **At most once** - Offsets are committed as soon as messages are received. If the processing goes wrong, some messages will be lost.
3. **Exactly once** - For Kafka use Transactional API (easy with Kafka Streams API), or use External Systems workflows: use an idempotent consumer