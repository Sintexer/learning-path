## Handler method

The Lambda function handler is the method in your function code that processes events. When your function is invoked, Lambda runs the handler method. When the handler exits or returns a response, it becomes available to handle another event. The handler method takes two objects – the **event object**  and the **context object (optional)**. 

###  Event

The **event** object differs in structure and contents, depending on which event source created it. 

### Context

The **context** object allows your function code to interact with the Lambda execution environment. At minimum context contains the elements:

- **AWS RequestID** – Used to track specific invocations.
- **Runtime** – The amount of time in milliseconds remaining before a function timeout.
- **Logging** – Information about which Amazon CloudWatch Logs stream your log statements will be sent.

### Best practices

- When designing and writing Lambda functions, it is best practice to separate the business logic from the handler method.
- It is a best practice to make your functions modular. For example, instead of having one function that does compression, thumb-nailing, and indexing, consider having three different functions that each serve a single purpose.
- Because your functions only exist when there is work to be done, it is particularly important for serverless applications to treat each function as stateless.
- Minimize both your deployment package dependencies and its size.  This can have a significant impact on the startup time for your function. For example, only choose the modules that you need — do not include an entire AWS SDK.

## Invocation

There are three ways to build and deploy your Lambda functions – the **Lambda console editor**, **deployment packages**, and **automation tools**.

### Lambda console editor

You can author functions within the Lambda console, with an IDE toolkit, using command line tools, or using the AWS SDKs. If you are new to Lambda, building functions in the console is the best place to begin writing your functions. If your code does not require custom libraries (other than the AWS SDK), you can edit your code inline through the console. When working with Lambda via the console, note that when you save your Lambda function, the Lambda service creates a deployment package that it can run. Once this deployment package is created, your function is deployed to the AWS Cloud. Because of this, you should build your functions using an account that is suitable for testing, and disable any selected triggers until your code testing is completed.

### Deployment packages

Your Lambda function's code consists of scripts or compiled programs and their dependencies. As developers increase their skills and advance beyond using the AWS Lambda console, they start using **deployment packages** to deploy the function code. Lambda supports two types of deployment packages – **container images** and **.zip file archives**. You can create and upload a .zip file to S3 or use a container image and push to [[AWS ECR]].

### Automate using tools

Serverless applications built using Lambda are a combination of Lambda functions, event sources, and other resources defined using the [[AWS SAM]]. You can automate the deployment process of your applications by using AWS SAM and other AWS services, such as [[AWS CodeBuild]], [[AWS CodeDeploy]], and [[AWS CodePipeline]].

### Best practices

- Include logging.
- Use environment variables to dynamically change code behavior.
- Use [[AWS Secrets Manager]] and [[AWS Secrets Manager#Parameters Store|Parameters Store]] for keys and secrets.
- Avoid recursion.
- Gather metrics with [[AWS CloudWatch]]
- Reuse execution context:
	1. Store dependencies locally.
	2. Limit re-initialization of variables.
	3. Reuse existing connections.
	4. Use tmp space as transient cache.
	5. Check that background processes have completed.
