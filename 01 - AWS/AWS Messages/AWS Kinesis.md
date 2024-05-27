AWS Kinesis is a [[AWS Managed Service|fully managed service]] for real-time processing of streaming data at a massive scale. It is part of the AWS ecosystem and is used to collect, process, and analyze real-time data.

> You can use Kinesis streaming services to ingest and process large volumes of data in near-real time.

When it comes to Kinesis, you need to understand the difference between producers and consumers.

**Producers** add data records to the stream. They can be created with Kinesis Producer Library (KPL), AWS SDK, or third-party tools. **Consumers** get records and process them. They can by Lambda functions, other streams, or Application created with Kinesis Client Library (KCL).

Kinesis Data Streams scales horizontally by adding **shards**. Each shard has a uniquely identified sequence of data records, and each data record has a sequence number assigned by Kinesis. The **Partition key** groups data by shard within a stream, the sequence number is unique per partition key within its shard. The following table compares the different rate and data limits for both writes and reads.

|                      | Writes  <br>(Putting Data on the Stream) | Reads  <br>(Consuming Data off the Stream)      |
| -------------------- | ---------------------------------------- | ----------------------------------------------- |
| **Rate Limit**       | 1,000 records per second                 | 5 transactions per second, up to 10,000 records |
| **Data Limit  <br>** | 1 MB per second                          | 2 MB per second                                 |

AWS Kinesis is capable of processing hundreds of terabytes of data per hour from high volume sources such as operational logs, IoT data streams, application logs, website clickstreams, and financial transactions. 

## AWS Kinesis Components

Kinesis consists of several components including [[AWS Kinesis Data Streams]], [[AWS Kinesis Data Firehose]], [[AWS Kinesis Data Analytics]], and [[Kinesis Video Streams]]. 

- **Kinesis Data Streams** capture and store data records from data producers for up to one year. 
- **Kinesis Data Firehose** captures, transforms, and loads data streams into AWS data stores for near real-time analytics with existing business intelligence tools.
- **Kinesis Data Analytics** processes and analyzes data streams in real time with SQL or Apache Flink.
- **Kinesis Video Streams** captures, processes, and stores video streams for analytics and machine learning.

See [[AWS Kinesis Data Streams vs Firehose]] comparison for a deeper comparison of two services.

## Use Cases

AWS Kinesis is used in a variety of real-time data streaming use cases, such as real-time dashboards, real-time analytics, IoT device data telemetry and metering, log and event data collection, and more.