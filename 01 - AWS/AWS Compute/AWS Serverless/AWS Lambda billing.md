With [[AWS Lambda]], you pay only for what you use. You are charged based on the **number of requests** for your functions and **the duration**. Lambda counts a request each time it starts running in response to an event notification or an invoke call, *including test invokes from the console*.

**Duration** is calculated from the time your code begins running until it returns or otherwise terminates, **rounded up to the nearest 1ms**. Price depends on the amount of memory you **_allocate_** to your function, not the amount of memory your function uses. If you allocate 10 GB to a function and the function only uses 2 GB, *you are charged for the 10 GB*. This is another reason to test your functions using different memory allocations to determine which is the most beneficial for the function and your budget. 

In the AWS Lambda resource model, you can choose the amount of memory you want for your function and are allocated proportional CPU power and other resources. *An increase in memory triggers an equivalent increase in CPU* available to your function. The AWS Lambda Free Tier includes 1 million free requests per month and 400,000 GB-seconds of compute time per month.

## The balance between power and duration

> [!success] Fact
> Depending on the function, you might find that the higher memory level might actually cost less because the function can complete much more quickly than at a lower memory configuration.

You can use an open-source tool called [[Lambda Power Tuning]] to find the best configuration for a function. The tool helps you to visualize and fine-tune the memory and power configurations of [[AWS Lambda function|Lambda functions]]. The tool runs in your own AWS account—powered by AWS Step Functions—and supports three optimization strategies: **cost**, **speed**, and **balanced**. It's language-agnostic so that you can optimize any Lambda functions in any of your languages. 

For deployment information and specifications on Lambda Power Tuning, see [aws-lambda-power-tuning](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:451282441545:applications~aws-lambda-power-tuning). This resource provides detailed instructions and explanations on running the tool.

