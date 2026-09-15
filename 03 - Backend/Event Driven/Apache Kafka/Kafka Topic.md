Topic is a *named* stream of data. It is like a table in the database. You can have as many topics as you need. Topic supports any kind of message format.

The sequence of messages in a topic is called a **data stream**.

You cannot query topics. Use [[Kafka Producer]] to send data and **Kafka Consumers** to read the data.