Event sources can invoke a Lambda function in three general patterns. These patterns are called invocation models. Each invocation model is unique and addresses a different application and developer needs. The invocation model you use for your Lambda function often depends on the event source you are using. It's important to understand how each invocation model initializes functions and handles errors and retries.

They are:

- **Synchronous invocation**
- **Asynchronous invocation**
- **Polling invocation**

## Synchronous invocation

> Request-response

When you invoke a function synchronously, **Lambda runs the function and waits for a response**. When the function completes, Lambda returns the response from the function's code with additional data, such as the version of the function that was invoked. Synchronous events expect an immediate response from the function invocation. 

With this model, there are no built-in retries. You must manage your retry strategy within your application code.

The following AWS services invoke Lambda synchronously:

- [[AWS API Gateway]]
- [[AWS Cognito]]
- [[AWS CloudFormation]]
- [[Amazon Alexa]]
- [[AWS Lex]]
- [[AWS CloudFront]]

## Asynchronous invocation

When you invoke a function asynchronously, events are **queued** and the **requestor doesn't wait** for the function to complete. This model is appropriate when the client *doesn't need an immediate response*. 

With the asynchronous model, you can make use of **destinations**. Use destinations to send records of asynchronous invocations to other services. A **destination** can send records of asynchronous invocations to other services. You can configure separate destinations for events that fail processing and for events that process successfully. You can configure destinations on a function, a version, or an alias, similarly to how you can configure error handling settings. With destinations, you can address errors and successes without needing to write more code.

The following AWS services invoke Lambda asynchronously: 

- [[AWS SNS]] 
- [[AWS S3]]
- [[AWS EventBridge]]

## Polling invocation

This invocation model is designed to integrate with AWS streaming and queuing based services with no code or server management. **Lambda will poll (or watch) these services**, retrieve any matching events, and invoke your functions. This invocation model supports the following services:

- [[AWS Kinesis]]
- [[AWS SQS]]
- [[AWS DynamoDB Stream]]

With this type of integration, AWS will manage the poller on your behalf and perform synchronous invocations of your function. With this model, the retry behavior varies depending on the event source and its configuration.

By default lambda uses 5 parallel processes to get messages from the queue. These five parallel processes mean Lambda is invoking **5 concurrent instances** of your Lambda function. To avoid your Lambda function getting throttled right out of the gate, make sure that the reserved concurrency on the function is at least 5.

> [!info]
> Best practice, is to set your SQS queue visibility timeout to 6 times the function timeout.

## Event source mapping

The configuration of services as event triggers is known as event source mapping. This process occurs when you configure event sources to launch your Lambda functions and then grant theses sources IAM permissions to access the Lambda function.

Lambda reads events from the following services:

- Amazon DynamoDB
- Amazon Kinesis
- Amazon MQ
- Amazon Managed Streaming for Apache Kafka (MSK)
- self-managed Apache Kafka
- Amazon SQS

## Invocation model error behavior  

When deciding how to build your functions, consider how each invocation method handles errors. The following chart provides a quick outline of the error handling behavior of each invocation model.

|Invocation model|Error behavior|
|---|---|
|Synchronous|No retries|
|Asynchronous|Built in – retries twice|
|Polling|Depends on event source|