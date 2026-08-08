
Kafka receives and outputs messages as bytes.

Because of that both [[Kafka Producer]] and [[Kafka Consumer]] must know Serialization and Deserialization details. The serialization/deserialization type must now change during a topic lifecycle (create a new topic instead).

## Kafka features

- Distributed
- Resilient
- Fault tolerant
- Horizontal scalability
- High performance
- Used by many high-end vendors, such as LinkedIn, Netflix, Uber, AirBnB