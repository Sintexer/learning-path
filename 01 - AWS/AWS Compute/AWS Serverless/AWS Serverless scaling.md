**Scaling best practices:**

- Separate your application and database.
- Take advantage of the AWS Global Cloud Infrastructure.
- Identify and **avoid heavy lifting**.
- Monitor for percentile.
- Refactor as you go.

In a traditional cloud approach with EC2 instances running the application you have to achieve high availability by introducing redundancy to your system, incorporate load balancers, configure VPCs and manage all of this. In a serverless approach you are building on top of scalable and redundant services with high availability, your infrastructure becomes much easier to maintain and develop.

With serverless, you don’t need to set up Auto Scaling groups and subnets because you’re using managed services that have built-in horizontal scaling, security, and high availability. 

## Considerations

To successfully scale your serverless architecture, you need to know the capabilities and service limits of the services that you’re integrating. Also, select patterns that optimize your application for the scale you need to support. Key considerations include the following:

- Timeouts
- Retry behaviors
- Throughput
- Payload size

## AWS API Gateway scaling considerations

[[AWS API Gateway]]. Thing to keep in mind is 

**Key questions**: Who will be using each API? Where will they be accessing it from, and how frequently? What are the downstream systems that could impact performance? Do you need to throttle requests at the "front door" to prevent overloading a downstream component?

**What can be done**: 

- You can configure the **optional cache** to reduce hits on your backend. 
- You could also set up **API keys and usage plans** to **throttle** request volume, client by client. 
- You can configure [[AWS API Gateway components#Endpoint type|Endpoint type]]: Regional or Edge optimized.

**Things to keep in mind**:

- [[AWS Lambda authorizers]] impact [[AWS Lambda function concurrency]] limit.
- API Gateway has a 10-MB payload limit, but the [[AWS SQS]] message size is limited to 256-KB.

## AWS SQS scaling consideration

[[AWS SQS]] buffers requests from [[AWS API Gateway]], enabling asynchronous processing. When used as an [[AWS Lambda invocation models#Polling invocation|AWS Lambda event source]], Lambda manages **polling** the queue, but you control key configuration options that impact performance.

**What can be done**:

- Change lambda function settings: batch size, concurrency limit, timeout limit (function run max time).
- Change SQS settings: visibility timeout (best practice it to make it at least 6x times lambda timeout), max receive count, redrive policy, dead-letter queue.

**Things to keep in mind**:

- Lambda uses 5 parallel processes to poll queue (by default), invoking five concurrent instances of you function. Ensure reserved concurrency is at least five to avoid throttling.
- Lambda increases the number of concurrent invocations as the queue size grows, up to a maximum of 1,000 (up to 60 per minute).
- If error occur, Lambda reduces the polling rate to alleviate pressure on downstream targets.
- Failed messages are retried up to the max receive count. After max receive count is breached, message is moved to the dead-letter queue.
- Too large batch size might lead to function timeout, too small leads to inefficient processing.

## AWS Lambda scaling considerations


**What can be done**:

- Update lambda [[AWS Lambda function concurrency#Concurrency Limits|concurrency limits]] to match potential traffic volume and limit max concurrency scenario.
- Update [[AWS Lambda function burst|lambda burst behavior]] to handle potential cases of traffic burst during peek hours.
- Update [[AWS Lambda function configuration#Memory|lambda memory]] configuration to make it cost efficient (you might use AWS Lambda Power Tuning tool) and match concurrency needs.

**Things to keep in mind**:

- Regularly review and adjust concurrency and memory settings based on performance and cost metrics.
- Implement a multi-account strategy to isolate environments and manage resources efficiently.
- Utilize AWS tools like AWS Organizations and AWS Control Tower for streamlined account management and security integration.
