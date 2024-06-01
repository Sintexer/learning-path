## Concurrency

Concurrency is the third major configuration that affects your function's performance and its ability to scale on demand. **Concurrency limits** help manage the number of simultaneous executions of your Lambda functions to avoid unexpected costs and performance issues.

- When your function is invoked, Lambda launches an instance of the function to process the event. 
- When the function code finishes running, it can handle another request. 
- If the function is invoked again while the first request is still being processed, another instance is allocated. 

Having more than one invocation running at the same time is the function's concurrency. For asynchronous event source, concurrency is measured as `Concurrency = request rate * average duration`. For stream invocations, there is a limit of one concurrent invocation per shard. As an analogy, you can think of concurrency as the total capacity of a restaurant for serving a certain number of diners at one time. Concurrency can be compared to the seating limit in a restaurant. If there are 100 total seats and 40 are in use and 20 are reserved, only 40 seats are available (40+20+40=100 seats) for people arriving without a reservation. If you have 25 requests per second and the function invocation takes 10 seconds to complete, you concurrency limit must be at least 250, otherwise, requests will be throttled.

Two primary impacts on concurrency are:

1. Invocation model
2. Service limits - which AWS service you use

[[AWS Lambda authorizers]] invocations are also count towards total concurrency, it is important to keep in mind.

## Concurrency Limits

- **Account Limits:** Protect against runaway functions that could incur high costs and impact downstream resources.
- **Function Limits:** Reserve part of your account limit for specific functions to ensure availability or prevent overwhelming downstream resources.
- **Emergency Brake:** Set function concurrency to 0 to stop all invocations immediately.
- **Load Testing:** Test all functions under concurrent usage scenarios to avoid conflicts.
- **Multi-Account Strategy:** Use AWS Organizations and AWS Control Tower to segregate environments and manage resources effectively.

## Concurrency types

Account-level concurrency limit is 1000 functions. Can be increased via a request.

**Unreserved concurrency**. The amount of concurrency that is not allocated to any specific set of functions. The minimum is 100 unreserved concurrency. This allows functions that do not have any provisioned concurrency to still be able to run. If you provision all your concurrency to one or two functions, no concurrency is left for any other function. Having at least 100 available allows all your functions to run when they are invoked.

**Reserved concurrency**. Guarantees the maximum number of concurrent instances for the function. When a function has reserved concurrency, no other function can use that concurrency. No charge is incurred for configuring reserved concurrency for a function.

**Provisioned concurrency**. Initializes a requested number of runtime environments so that they are prepared to respond immediately to your function's invocations. This option is used when you need high performance and low latency. You pay for the amount of provisioned concurrency that you configure and for the period of time that you have it configured. For example, you might want to increase provisioned concurrency when you are expecting a significant increase in traffic. To avoid paying for unnecessary warm environments, you scale back down when the event is over.

![[AWS Lambda function concurrency types.png]]

Limit a function’s concurrency to achieve the following:

- Limit costs
- Regulate how long it takes you to process a batch of events
- Match it with a downstream resource that cannot scale as quickly as Lambda

Reserve function concurrency to achieve the following: 

- Ensure that you can handle peak expected volume for a critical function 
- Address invocation errors

## Concurrency bursts

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