Producers write data to topics. Producers know to which partition to write to (and which Kafka broker has it), ahead of write.

In case of Kafka broker failures, Producers will automatically recover.