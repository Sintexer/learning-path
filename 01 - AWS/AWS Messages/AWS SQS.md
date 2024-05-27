> AKA **Simple Queue Service**

AWS SQS is a fully-managed message queuing service. It is used to decouple and scale microservices, distributed systems, and serverless applications. SQS eliminates the complexity and overhead associated with managing and operating message-oriented middleware, allowing developers to focus on differentiating work. It can be utilized to create a queue of messages with the delayed processing feature, facilitating communication between services in a distributed systems setup.


In SQS, messages can be up to `256 KB` in size, and they are stored in the queue until they are processed by a consumer, which reads and then deletes a message from the queue. The visibility timeout feature in SQS prevents other consumers from processing the message again. SQS also supports **dead-letter queues** for messages that can't be processed. 

SQS provides robust **security** for your applications. You can control who can send messages to a queue (Producers) and who can receive messages from a queue (Consumers) using [[IAM]]. SQS guarantees the delivery of messages between producers and consumers and allows for high scalability and elasticity with no provisioning required.

## Queue types

There are two types of message queues that AWS SQS offers: 
1. Standard Queues
2. FIFO (First-In-First-Out) Queues. 

### Standard SQS

**Standard Queues** offer maximum throughput, best-effort ordering, and at-least-once delivery. They have an unlimited throughput and *the order of messages isn't guaranteed*, but they are generally delivered in the same order as they were sent. 

- **Record Order**: The order of messages is *not guaranteed*. Messages are generally delivered in the same order they are sent, but there might be exceptions.
- **Delivery**: Messages may be delivered more than once. Multiple copies of the same message might end up in the queue.
- **Transaction Throughput**: Standard SQS supports nearly unlimited messages per second. It can handle a high volume of transactions.

### FIFO SQS

**FIFO Queues** provide the exact order of messages, i.e., *precisely-once processing*. They are designed to assure that the *order of the messages is maintained*, ensuring that the first message entered is the first one to be processed.

- **Record Order**: The order of messages *is guaranteed* per group ID. Messages are delivered in the exact order they are sent.
- **Delivery**: Messages are delivered only once. There are no duplicate messages introduced to the queue. This ensures that processing tasks aren't duplicated.
- **Transaction Throughput**: FIFO queues support up to 300 messages per second per API action without batching, or 3,000 messages per second with batching. This maintains the order while still providing substantial throughput.
## Integrations

SQS offers [[AWS SQS Lambda integration|easy integration with Lambda]] and other AWS services such as [[AWS S3]],, [[EC2]], [[AWS SNS]], and more, making it a fundamental building block for many applications on the AWS platform. Understanding SQS is essential for one's AWS knowledge stack, particularly in managing serverless architectures, microservices, distributed systems, and decoupling applications.