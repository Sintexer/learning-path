The key idea of AWS Serverless architecture is to integrate your frontend with the API (usually it's [[AWS API Gateway]]), add a [[AWS SQS|queue]] for incoming traffic from API to deal with burst load and to store requests durably, and then pass these messages to [[AWS Lambda]] to process the request. Al of that is processed asynchronously thanks to the queue. When client makes a request, it receives a message id, that message id is related to the response that will be provided by your service in future. This queue + lambda pattern is described in [[AWS SQS Lambda integration]] article.

In an architecture described above, the client service submitted a request, and the SQS queue gave a response to API Gateway with the message ID. Then Lambda pulled the message from the queue and handed it and update the job status. The next thing you need to consider is how to provide the status to the calling client. The method you choose depends on your use case. The next set of videos looks at three potential approaches:

- Client polling
- Webhooks with [[AWS SNS]]
- WebSockets with [[AWS AppSync]]

###  AWS Client polling pattern  

![[Client polling pattern]]

In the context of AWS, once the client sends a message to an SQS queue to start a long-running transaction, it receives a **message ID**. The client can then use this ID to poll an API Gateway endpoint, which in turn triggers a Lambda function. This function fetches the status of the request from a persistent storage where the status is being updated by another Lambda function processing the SQS messages.

While this pattern is simple to implement and easy to understand, it does have some downsides:

- **Additional network traffic**. It can lead to unnecessary network traffic
- **Costly**. it can increase costs if the polling frequency is high. 
- **Increased latency**. It can also lead to increased latency in getting the final result as the client might poll when no update is available. To mitigate this, strategies like exponential backoff can be used to increase the delay between polls over time.

## Webhook pattern with SNS

The webhook pattern with [[AWS SNS]] is a powerful architectural pattern for real-time, event-driven, serverless workflows. It allows services to communicate with each other through HTTP callbacks, providing a way to deliver data to other applications as soon as it becomes available.

In the context of AWS, a webhook can be implemented by publishing messages to an [[SNS topic]]. SNS is a fully managed pub/sub messaging service that can fan out messages to a large number of subscribers, which can be [[AWS Lambda function|AWS Lambda functions]], Amazon SQS queues, HTTP endpoints, and more. In terms of retry approaches, **SNS provides automatic retries**.

When an event occurs (like a file upload to an S3 bucket or a change in a DynamoDB table), a message is published to an SNS topic. This message can then trigger an `HTTP POST` request to a subscriber endpoint - **the webhook**. The webhook, which is simply a script on a web server that receives these POST requests, can then process the data in the message as needed.

This pattern is particularly useful in serverless architectures for real-time notifications and updating remote systems. For example, if you have an application where you want to notify users of a new post, you can publish the post details to an SNS topic, which then sends the data to a webhook URL associated with each user.

However, it's important to ensure the receiving endpoint of the webhook is secure and can validate the incoming requests, as it can be exposed to the public internet. AWS provides features like message filtering and encryption to help secure your SNS topics and messages.

In the context of the webhook pattern with Amazon SNS, the clients can be categorized as private or public based on their accessibility.

**Private Clients:** These are the services or systems within your private network or VPC. They are not exposed to the public internet. An example could be a microservice in your application architecture that needs to be notified when an event occurs.

- **Pros:** Enhanced security
- **Cons:** They can't be accessed directly by SNS as they are not publicly accessible. To overcome this, you can use services like [[AWS PrivateLink]] to allow SNS to access your private endpoint within your VPC.

**Public Clients:** These are the services or systems that are exposed to the public internet. An example could be a third-party application that subscribes to your SNS topic.

- **Pros:** Easy to set up and can be accessed directly by SNS.
- **Cons:** They are exposed to the public internet, which can be a security concern. They need to validate and authenticate the incoming requests to ensure they are from the expected SNS topic.

The webhook pattern differs from the [[#AWS Client polling pattern|polling pattern]] in the way they get updates. In the polling pattern, the client continuously asks or 'polls' the server for updates. In the webhook pattern, the server 'pushes' the updates to the client as soon as they are available.

## WebSockets pattern with AppSync

The last option you can use is **WebSockets** with [[AWS AppSync]]. WebSocket APIs are an open standard used to create a persistent connection between the client and the backing service, permitting bidirectional communication. You can provide the status for the client using WebSockets with [[GraphQL]] subscriptions to listen for updates through AWS AppSync.

In this pattern, the client establishes a WebSocket connection with AWS AppSync. This connection is a persistent, *two-way communication channel*. When a data change occurs on the server-side, it's instantly pushed to the client over the WebSocket connection.

This pattern is particularly useful for applications that require real-time collaboration or instant updates, such as chat applications, collaborative editing tools, or real-time analytics dashboards.

Key considerations for this pattern include:

- **Real-time updates**: This pattern provides real-time updates and two-way communication between the client and server.
- **Persistent connections**: WebSocket connections are persistent, which can lead to higher resource usage on the server-side.
- **Complexity**: Managing WebSocket connections and handling connection errors can add complexity to the application. AWS AppSync handles this complexity, making it easier to build real-time applications.

