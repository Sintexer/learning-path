When you are ready to make your [[AWS API Gateway]] callable for your users, you need to deploy your API to a **stage**.

A stage is a **snapshot of the API** and represents a unique identifier for a version of a deployed API. With stages, you can have multiple versions and roll back versions. 

> [!info]
> Anytime you update anything about the API, you need to redeploy it to an existing stage or to a new stage that you create as part of the deploy action.

Stage is used in [[AWS API Gateway components#Invoke URL|API Invoke URL]].

Critical design options are set per stage including: 

- Caching
- Throttling
- Usage plans

At an API stage, you can also export the API definitions or generate an SDK for your users to call the API using a supported programming language.

Some options for you to differentiate your APIs with stages include:

- Use different stages by environment or customer.
- Use stage variables to increase deployment flexibility.
- Use stages with canary deployments to test new versions.


## Stage variable

As you define variables in the stage settings in the console, you can reference them with the `$stageVariables.[variable name]` notation. You can also inject stage-dependent items at runtime such as:

- URLs
- [[AWS Lambda function|Lambda functions]]
- Any necessary variables

This is a great way for you to perform actions such as invoking different endpoints and different backends for different stages. For example, if you have different endpoints with different URLs or different Lambda function aliases based on environment, you can use stage variables to inject those variables.

> [!example]
> You can add variables such as `lambdaFn`, and `url` to then reference them in integration configuration like `${stageVariables.lambdaFn}` and `${stageVariables.url}`.

It is very useful for automated CI/CD.