## Lambda function

The **Lambda function** is the foundational principle of **AWS Lambda**. You have the option of configuring your functions using the **Lambda console**, **Lambda API**, [[AWS CloudFormation]], or [[AWS Serverless Application Model]]. You can invoke your function directly by using the Lambda API, or you can configure an AWS service or resource to invoke your function in response to an event.

A **function** is a resource that you can invoke to run your code in Lambda. Lambda *runs instances* of your function to process events. When you create the Lambda function, it can be authored in several ways:

- You can create the function from scratch.
- You can use a blueprint that AWS provides.
- You can select a container image to deploy for your function.
- You can browse the [[AWS Serverless Application Repository]].

### Trigger

**Triggers** describe **when** a Lambda function should run. A trigger integrates your Lambda function with other AWS services and event source mappings. So you can run your Lambda function in response to certain API calls or by reading items from a stream or queue. This increases your ability to respond to events in your console without having to perform manual actions.

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