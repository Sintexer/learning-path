[[AWS Lambda]] automatically monitors Lambda functions on your behalf and reports metrics through [[AWS CloudWatch]]. To help you monitor your code when it runs, Lambda automatically tracks the following:

- Number of requests
- Invocation duration per request
- Number of requests that result in an error

Amazon CloudWatch provides built-in metrics to help monitor your Lambda functions: **Invocations**, **Duration**, **Errors**, **Throttles** (failures because of concurrency limits.), **IteratorAge**, **DeadLetterErrors**, **ConcurrentExecutions**.

You must also rely on **Lambda Dead letter queues** - they are the queues of lambda failure events. You might even configure [[AWS SNS]] to send events on lambda failures to address them asap.

## Amazon CloudWatch Lambda Insights

**CloudWatch Lambda Insights** is a monitoring and troubleshooting solution for serverless applications running on Lambda. Lambda Insights collects, aggregates, and summarizes system-level metrics. It also summarizes diagnostic information such as cold starts and Lambda worker shutdowns to help you isolate issues with your Lambda functions and resolve them quickly.

Lambda Insights uses a new CloudWatch Lambda extension, which is provided as a Lambda layer. When you enable this extension on a Lambda function, it collects system-level metrics and emits a single performance log event for every invocation of that Lambda function. CloudWatch uses **embedded metric formatting (EMF)** to extract metrics from the log events.

## Monitoring Lambda functions using [[AWS X-Ray]]

You can use AWS X-Ray to visualize the components of your application, identify performance bottlenecks, and troubleshoot requests that resulted in an error. Your Lambda functions send trace data to X-Ray, and X-Ray processes the data to generate a service map and searchable trace summaries.

AWS X-Ray records how the Lambda functions are running.  

You can use X-Ray for:

- Tuning performance
- Identifying the call flow of Lambda functions and API calls
- Tracing path and timing of an invocation to locate bottlenecks and failures

## Monitoring Lambda functions using [[AWS CloudTrail]]

AWS CloudTrail helps audit your application by recording all the API actions made against the application. These logs can be exported to the analysis tool of your choice for additional analysis. 

CloudTrail logging provides the following options:

- The default Lambda CloudTrail logging is for control plane (management) events.
- Optional logging: CloudTrail also logs data events. You can turn on data event logging so that you log an event every time Lambda functions are invoked.

CloudTrail can be an important tool for auditing serverless deployments and rolling back unplanned deployments.