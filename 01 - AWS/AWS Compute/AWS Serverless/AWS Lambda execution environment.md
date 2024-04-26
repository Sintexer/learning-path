[[AWS Lambda]] invokes your function in an **execution environment**, which is a secure and isolated environment. The execution environment manages the resources required to run your function. The execution environment also provides lifecycle support for the function's runtime and any external extensions associated with your function.

When you write your function code, **do not assume that Lambda automatically reuses the execution environment** for subsequent function invocations. Other factors may require Lambda to create a new execution environment, which can lead to unexpected results. Always test to optimize the functions and adjust the settings to meet your needs.

![[AWS Lambda execution environment diagram.png]]

## Init phase

In this phase, Lambda *creates or unfreezes an execution environment* with the configured resources, *downloads the code* for the function and all layers, *initializes any extensions*, *initializes the runtime*, and then *runs the function’s initialization code* (the code outside the main handler). 

The Init phase happens either during the first invocation, or before function invocations if you have enabled provisioned concurrency.

The Init phase is split into three sub-phases: 

1. **Extension init** - starts all extensions
2. **Runtime init** - bootstraps the runtime
3. **Function init** - runs the function's static code

These sub-phases ensure that all extensions and the runtime complete their setup tasks before the function code runs.

The Init phase ends when the runtime and all extensions signal that they are ready by sending a `Next` API request. The `Init` phase is limited to 10 seconds. If all three tasks do not complete within 10 seconds, Lambda retries the `Init` phase at the time of the first function invocation with the configured function timeout.

When [Lambda SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html) is activated, the `Init` phase happens when you publish a function version. Lambda saves a snapshot of the memory and disk state of the initialized execution environment, persists the encrypted snapshot, and caches it for low-latency access. If you have a `beforeCheckpoint` [runtime hook](https://docs.aws.amazon.com/lambda/latest/dg/snapstart-runtime-hooks.html), then the code runs at the end of `Init` phase.

AWS is responsible for optimizing the time required to initialize environment and runtime. Billing starts on packages and dependencies initialization, so it up to user to optimize this part.

## Invoke phase

In this phase, Lambda invokes the function handler. After the function runs to completion, Lambda **prepares to handle another function invocation**.

## Shutdown phase

If the Lambda function does not receive any invocations for a period of time, this phase initiates. In the Shutdown phase, Lambda *shuts down the runtime*, *alerts the extensions* to let them stop cleanly, and then *removes the environment*. Lambda sends a shutdown event to each extension, which tells the extension that the environment is about to be shut down.

## Provisioned concurrency

If you need predictable function start times for your workload, provisioned concurrency ensures the lowest possible latency. This feature keeps your functions initialized and warm, and ready to respond in double-digit milliseconds at the scale you provision. Unlike with on-demand Lambda, this means that all setup activities happen before invocation, including running the initialization code.

A function with a provisioned concurrency of 6 has 6 runtime environments prepared before the invocations occur. In the time between initialization and invocation, the runtime environment is prepared and ready.

## Best practices

1. Store and reference dependencies locally.
2. Limit re-initialization of variables.
3. Add code to check for and reuse existing connections.
4. Use tmp space as transient cache.
5. Check that background processes have completed.