AWS Kinesis Data Streams is a scalable and durable real-time data streaming service that can continuously capture gigabytes of data per second from hundreds of thousands of sources.

Key Features:

- **Scalability** - a major feature of Kinesis Data Streams. It can handle streaming data at a massive scale, from a few kilobytes to terabytes per hour, and from a few to millions of PUT records per second.
- **Durability** - Kinesis Data Streams stores the data redundantly across multiple facilities and allows the data to be available for processing for up to one year.
- **Real-Time Data Processing** - it enables you to collect and process data in real-time, allowing you to analyze and respond to the incoming data instantly.
- **Integration with AWS Services** -  it integrates with other AWS services like [[AWS Lambda]] for custom data processing and analysis.

The main alternative of AWS Kinesis Data Streams is [[AWS Kinesis Data Firehose]]. Firehouse offers a simpler management experience and autoscaling, but lacks flexibility in terms of configuration. See [[AWS Kinesis Data Streams vs Firehose]] comparison for a deeper comparison of two services.

Kinesis Data Streams is used in various real-time data streaming use cases. For instance, it can be used for real-time metrics and reporting, log and event data collection, real-time data analytics, and more.