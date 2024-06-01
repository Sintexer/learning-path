AWS Lambda Authorizers are a feature of [[AWS API Gateway]] that allow you to use [[AWS Lambda function|AWS Lambda functions]] to control access to your APIs. When a client makes a request to your API, the Lambda authorizer function is invoked to **validate the request**, typically by checking tokens, headers, or other request parameters. Based on the logic you define, the Lambda function can allow or deny access by returning an [[IAM policy]]. 

There are two types of Lambda authorizers: 

1. **Token-based**: Validates a token (e.g., [[JWT]]) passed in the request header.
2. **Request-based**: Validates request parameters, headers, or other data.

This provides a flexible and customizable way to implement authentication and authorization for your APIs.

## Concurrency considerations

Authorizer function invocations count towards total [[AWS Lambda function concurrency|concurrency]]. Anticipate authorization request volume to avoid exceeding concurrency limits. Cache authorizations for 5 minutes to an hour to reduce function invocations.