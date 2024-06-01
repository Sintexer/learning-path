When building and testing a [[AWS Lambda function]], you must specify three primary configuration settings: **memory**, **timeout**, and **concurrency**. These settings are important in defining how each function performs. Deciding how to configure memory, timeout, and concurrency comes down to testing your function in real-world scenarios and against peak volume. As you monitor your functions, you must adjust the settings to optimize costs and ensure the desired customer experience with your application.

## Memory

You can allocate up to 10 GB of memory to a Lambda function. Lambda **allocates CPU** and other resources linearly in proportion to the amount of memory configured. Any increase in memory size triggers an equivalent increase in CPU available to your function. To find the right memory configuration for your functions, use the [AWS Lambda Power Tuning tool](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:451282441545:applications~aws-lambda-power-tuning).

Because Lambda charges are proportional to the memory configured and function duration (GB-seconds), the additional costs for using more memory may be offset by lower duration.

## Timeout

The AWS Lambda timeout value dictates how long a function can run before [[AWS Lambda]] terminates the Lambda function. At the time of this publication, the maximum timeout for a Lambda function is 900 seconds. This limit means that a single invocation of a Lambda function cannot run longer than 900 seconds (which is 15 minutes). 

Set the timeout for a Lambda function to the maximum only after you test your function. There are many cases when an application should fail fast and not wait for the full timeout value. 

It is important to analyze how long your function runs. When you analyze the duration, you can better determine any problems that might increase the invocation of the function beyond your expected length. Load testing your Lambda function is the best way to determine the optimum timeout value.

> [!note] Pricing is based on time
Your Lambda function is billed based on runtime in **1-ms increments**. Avoiding lengthy timeouts for functions can prevent you from being billed while a function is simply waiting to time out. [[AWS Lambda billing|AWS Lambda billing is explained here]].

## Concurrency

Concurrency is the number of Lambda invocation that can be run at the same time. If the number of requests for invocations exceeds the account or Lambda function concurrency limit, requests are throttled. See more at [[AWS Lambda function concurrency]].