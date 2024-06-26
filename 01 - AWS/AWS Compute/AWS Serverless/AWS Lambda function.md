The **Lambda function** is the foundational principle of **[[AWS Lambda]]**. Think of a function as a small, self-contained application lambdas are stateless and each function is associated with code and some configuration information. You have the option of configuring your functions using the **Lambda console**, **Lambda API**, [[AWS CloudFormation]], or [[AWS Serverless Application Model]]. You can invoke your function directly by using the Lambda API, or you can configure an AWS service or resource to invoke your function in response to an event. Lambda is integrated with [[AWS CodeDeploy]] for automated rollout with traffic shifting.

A **function** is a resource that you can invoke to run your code in Lambda. Lambda *runs instances* of your function to process events. When you create the Lambda function, it can be authored in several ways:

- You can create the function from scratch.
- You can use a blueprint that AWS provides.
- You can select a container image to deploy for your function.
- You can browse the [[AWS Serverless Application Repository]].

You can configure Lambda with a set of permissions and define what services it is permitted to interact with using [[IAM]]. Also you need to configure which events or event sources can initiate the function. Assign a code to execute, and at the end configure execution parameters: [[AWS Lambda function configuration|memory, timeout and concurrency]]. Different configurations impact on [[AWS Lambda billing]].

One potential risk is every change to Lambda is published and visible in the system. It might be hard to rollback if something is wrong. This problem might be solved by [[AWS Lambda function versioning]].

## Burst behavior

Where there is a situation when the number of requests spikes immediately, lambda uses [[AWS Lambda function burst]] behavior to handle the increased number of invocations.

## Processing error

For an asynchronous event source, there are **two retries** for a failed or throttled request. For a synchronous event source there is **no retries built in**.

### Trigger

**Triggers** describe **when** a Lambda function should run. A trigger integrates your Lambda function with other AWS services and event source mappings. So you can run your Lambda function in response to certain API calls or by reading items from a stream or queue. This increases your ability to respond to events in your console without having to perform manual actions.

Event sources can invoke a Lambda function in three general patterns. These patterns are called **invocation models**. They are: [[AWS Lambda invocation models]]. You must choose which of them fulfills your requirements.

### Event

An **event** is a JSON-formatted document that **contains data** for a Lambda function to process. The runtime converts the **event to an object** and passes it to your function code. When you invoke a function, you determine the structure and contents of the event.

### Function environment and deployment 

An application environment provides a secure and isolated runtime environment for your Lambda function. An application environment manages the **processes** and **resources** that are required to run the function.

You deploy your Lambda function code using a deployment package. Lambda supports two types of deployment packages:

- **A .zip file archive** – This contains your function code and its dependencies. Lambda provides the *operating system and runtime* for your function.
- **A container image** – This is compatible with the [[Open Container Initiative|Open Container Initiative (OCI)]] specification. You add your *function code and dependencies to the image*. You must also *include the operating system and a Lambda runtime.*

### Function runtime

The runtime provides a language-specific environment that runs in an application environment. When you create your Lambda function, you specify **the runtime** that you want your code to run in. You can use built-in runtimes, such as **Python**, **Node.js**, **Ruby**, **Go**, **Java**, or **.NET Core**. Or you can implement your Lambda functions to run on a **custom runtime**.

### Function handler

The AWS Lambda function handler is the method in your function code that processes events. When your function is invoked, Lambda runs the handler method. When the handler exits or returns a response, it becomes available to handle another event. 

You can use the following general syntax when creating a function handler in Python.

```python
def _handler_name_ (event, context):

...

return _some_value_
```