By introducing the [[AWS SQS]] queue in front of [[AWS Lambda]], you create an **asynchronous** connection between the API request and Lambda. This allows you to satisfy the API request without regard for how long the Lambda function (or other backend services) will run.

**Scaling**. One key benefit of this integration is automatic scaling. AWS Lambda can automatically scale the number of instances running your function to process multiple SQS messages concurrently. You have control over the concurrency level by setting reserved concurrency limits to determine how many simultaneous invocations your function can have. This is particularly beneficial for bursts of traffic and high-volume processing workloads.

**Retry**. Another significant advantage is the built-in retry mechanism. Lambda will automatically retry the failed processing due to issues like timeouts or exceptions, meaning your messages will remain in the queue until they are processed successfully or the message's retention period expires. Moreover, SQS also provides a **dead-letter queue (DLQ)** where Lambda can send messages it can't process, for eventual debugging or reprocessing.

**Avoid duplicates**. Lambda's support for SQS FIFO queues is a notable feature. This ensures the order is maintained in processing and there's **no duplication of messages**. Your Lambda function can maintain the order of related messages and multiple messages can be batched together, reducing Lambda invocations and thus, the cost.

**Monitoring**. Integration with [[AWS CloudWatch]] for monitoring is almost seamless, delivering messages about the function’s performance and operation. This gives you a continuous view and insights into how your system operates.

**Security**. From a security perspective, Lambda integrates with [[IAM]], giving you full control over user permissions, and ensuring that your Lambda function and SQS queue are only accessed by authorized entities.

Sometimes logic behind one SQS message becomes too gib to be handled by one Lambda function. And in this case chain of calls from one Lambda to another grows in depth dramatically. Also, it leads to a hard-to-support code where each lambda must be aware of all other functions surrounding it in order to handle failure. For such advanced pipelines AWS has introduced [[AWS Step Functions]]