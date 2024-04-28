When building and testing a [[AWS Lambda function]], you must specify three primary configuration settings: **memory**, **timeout**, and **concurrency**. These settings are important in defining how each function performs. Deciding how to configure memory, timeout, and concurrency comes down to testing your function in real-world scenarios and against peak volume. As you monitor your functions, you must adjust the settings to optimize costs and ensure the desired customer experience with your application.

## Memory

You can allocate up to 10 GB of memory to a Lambda function. Lambda allocates CPU and other resources linearly in proportion to the amount of memory configured. Any increase in memory size triggers an equivalent increase in CPU available to your function. To find the right memory configuration for your functions, use the [AWS Lambda Power Tuning tool](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:451282441545:applications~aws-lambda-power-tuning).

Because Lambda charges are proportional to the memory configured and function duration (GB-seconds), the additional costs for using more memory may be offset by lower duration.

## Timeout

The AWS Lambda timeout value dictates how long a function can run before [[AWS Lambda]] terminates the Lambda function. At the time of this publication, the maximum timeout for a Lambda function is 900 seconds. This limit means that a single invocation of a Lambda function cannot run longer than 900 seconds (which is 15 minutes). 

Set the timeout for a Lambda function to the maximum only after you test your function. There are many cases when an application should fail fast and not wait for the full timeout value. 

It is important to analyze how long your function runs. When you analyze the duration, you can better determine any problems that might increase the invocation of the function beyond your expected length. Load testing your Lambda function is the best way to determine the optimum timeout value.

> [!note] Pricing is based on time
Your Lambda function is billed based on runtime in **1-ms increments**. Avoiding lengthy timeouts for functions can prevent you from being billed while a function is simply waiting to time out. [[AWS Lambda billing|AWS Lambda billing is explained here]].

## Concurrency

Concurrency is the third major configuration that affects your function's performance and its ability to scale on demand. Concurrency is the number of invocations your function runs at any given moment. 

- When your function is invoked, Lambda launches an instance of the function to process the event. 
- When the function code finishes running, it can handle another request. 
- If the function is invoked again while the first request is still being processed, another instance is allocated. 

Having more than one invocation running at the same time is the function's concurrency. As an analogy, you can think of concurrency as the total capacity of a restaurant for serving a certain number of diners at one time. Concurrency can be compared to the seating limit in a restaurant. If there are 100 total seats and 40 are in use and 20 are reserved, only 40 seats are available (40+20+40=100 seats) for people arriving without a reservation.

### Concurrency types

Account-level concurrency limit is 1000 functions. Can be increased via a request.

**Unreserved concurrency**. The amount of concurrency that is not allocated to any specific set of functions. The minimum is 100 unreserved concurrency. This allows functions that do not have any provisioned concurrency to still be able to run. If you provision all your concurrency to one or two functions, no concurrency is left for any other function. Having at least 100 available allows all your functions to run when they are invoked.

**Reserved concurrency**. Guarantees the maximum number of concurrent instances for the function. When a function has reserved concurrency, no other function can use that concurrency. No charge is incurred for configuring reserved concurrency for a function.

**Provisioned concurrency**. Initializes a requested number of runtime environments so that they are prepared to respond immediately to your function's invocations. This option is used when you need high performance and low latency. You pay for the amount of provisioned concurrency that you configure and for the period of time that you have it configured. For example, you might want to increase provisioned concurrency when you are expecting a significant increase in traffic. To avoid paying for unnecessary warm environments, you scale back down when the event is over.

![[AWS Lambda function configuration concurrency types.png]]

Limit a function’s concurrency to achieve the following:

- Limit costs
- Regulate how long it takes you to process a batch of events
- Match it with a downstream resource that cannot scale as quickly as Lambda

Reserve function concurrency to achieve the following: 

- Ensure that you can handle peak expected volume for a critical function 
- Address invocation errors

### Concurrency bursts

A burst is when there is a sudden increase in the number of instances needed to fulfill the requested number of running functions. An example is an increase in orders on a website during a limited time sale. The burst concurrency quota is not per function. It applies to all of your functions in the Region. 

The burst quotas differ by region:

- 3000 – US West (Oregon), US East (N. Virginia), Europe (Ireland)
- 1000 – Asia Pacific (Tokyo), Europe (Frankfurt), US East (Ohio)    
- 500 – Other Regions

After the initial burst, your functions' concurrency can scale by an additional 500 instances each minute. This continues until there are enough instances to serve all requests, or until a concurrency limit is reached.

## Testing concurrency

The most important factor for your concurrency, memory, and timeout settings is to verify application testing against real-world conditions. To do this, follow these suggestions:

- Run performance tests that simulate peak levels of invocations.
    - View the metrics for the amount of throttling that occurs during performance peaks.
- Determine whether the existing backend can handle the speed of requests sent to it.
    - Don't test in isolation. If you’re connecting to Amazon Relational Database Service (Amazon RDS), ensure that you test that the concurrency levels for your function can be processed by the database.
- Does your error handling work as expected? 
    - Tests should include pushing the application beyond the concurrency settings to verify correct error handling.