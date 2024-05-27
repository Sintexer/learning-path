
[[AWS Kinesis Data Streams]] and [[AWS Kinesis Data Firehose]] are two powerful services in the [[AWS Kinesis]] ecosystem for handling real-time data streams. While they share some similarities, they have distinct features and use cases that make them suitable for different scenarios.

## Flexibility and Complexity

With **Kinesis Data Streams**, you can write *custom consumers* and aren't limited to the targets that Kinesis Data Firehose supports. This gives you greater flexibility but adds more complexity to your setup. On the other hand, **Kinesis Data Firehose** simplifies the use case of transforming and storing streaming data. If Data Firehose meets your requirements, it's going to be an efficient choice for transforming and storing your data.

## Data Delivery

One critical difference is that Kinesis Data Streams guarantees *order delivery* of data records in the shard as well as *exactly-once delivery* of records, whereas Kinesis Data Firehose does not. Because Kinesis Data Streams must preserve the sequence of records, if a Lambda function fails on one record in a batch, the entire batch (and thus the shard) is blocked until either the message processes successfully or the retention period for the data records in the batch expires.

Kinesis Data Firehose handles data delivery and delivery failures differently depending on the consumer you’ve configured. When you associate a Lambda function to a Data Firehose stream to transform data on the stream, Data Firehose tries the invocation three times and then skips that batch of records. Records that failed to process are delivered to an Amazon S3 bucket in a *processing_failed* folder.

## Shard Management

You control the number of shards with **Kinesis Data Streams**, whereas with **Kinesis Data Firehose** the number of shards is managed by the service based on data volume.

## Consumers

A **Kinesis Data Firehose** stream is associated with a single destination, specified when you set up the stream, although you can configure an [[AWS S3 Bucket]] to receive a backup of source records. A **Kinesis data stream** can have multiple consumers and multiple types of consumers. By default, all the consumers on a shard share the read throughput of that shard, and there is a limit of five consumers per shard. If you add an *enhanced fan-out consumer*, it gets dedicated throughput, enabling you to support more than five consumers. But there is an additional cost for each enhanced fan-out consumer.

## What to choose?

**When to choose Kinesis Data Streams:**

- You need to write *custom consumers* that aren't limited to the targets that Kinesis Data Firehose supports. This could include AWS Lambda functions, Kinesis Data Analytics, or custom applications using the Kinesis Client Library.
- You require *guaranteed order* of message delivery within each shard.
- You require *[[Exactly-once delivery|exactly-once processing]]* of messages.
- You need *precise control* over streams processing configuration and performance, including the ability to manage the number of shards yourself.
- You need to configure *multiple consumers* for your data, including standard and enhanced fan-out consumers.

**When to choose Kinesis Data Firehose:**

- You primarily need to *load data* into AWS data stores (like S3, Redshift, or Elasticsearch) or to send data to Splunk, and you don't require custom data processing or consumers.
- You prefer a *simplified management* experience, where AWS handles the scaling and throughput management.
- You're okay with *[[At-least-once delivery|at-least-once delivery]]* semantics, where a message might be delivered and processed more than once in case of retries.
- You want built-in support for *data transformation* and *conversion* via [[AWS Lambda]] or [[AWS Glue]].
- You're okay with failed records being automatically stored in an S3 bucket for further analysis or reprocessing.