API Gateway is a service that facilitates the creation, publishing, maintenance, monitoring, and security of your [[API|APIs]] at any scale. API Gateway handles all your tasks around accepting and processing hundreds of thousands of concurrent API calls. This includes:

- [[AWS API Gateway access management|handling access and traffic management]]
- [[AWS API Gateway throttling|throttling]]
- monitoring
- [[AWS API Gateway stage|API version management]].

> [!example] **Example: Social identity provider API**
> Authenticating into an application using social identity providers such as Amazon, Facebook, or Google. When you select this option while logging in, the application will communicate with the social media provider's API to authenticate you using your identity provider account.

It is beneficial to follow [[AWS API Gateway best-practices]].

## Architecture

Usually a request to API Gateway starts from a mobile, web app or some API service. They access though HTTPS and can reach [[AWS CloudFront]] thank to how API Gateway is designed. Gateway first tries to fulfill the request using the response cache, otherwise it pass request to the [[🗂️ AWS Compute]], any publicly accessible endpoint, or some [[AWS VPC]].

API Gateway comes in three forms:

- [[AWS REST API Gateway]] - are intended for APIs that require API proxy functionality and API management features in a single solution.
- [[AWS HTTP API Gateway]] - are optimized for building APIs that proxy to Lambda functions or HTTP backends, making them ideal for serverless workloads. Cheaper and faster compared to REST APIs.
- [[AWS WebSocket API Gateway]]

**For a more in-depth comparison between HTTP and REST APIs, see** [Choosing between HTTP APIs and REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html).

When creating an API through the console, you'll have the option to create a new API, clone an existing API, or import an API as your starting point.

API gateway requires you to [[AWS API Gateway components|configure API name, resources, and resource methods in order to configure the API]]. You can also set-up [[AWS API Gateway integration types|different integrations]] with other AWS services.

Once you’ve saved the details of the integration type, the console will display panels where you can add to your request-response details. For example, you might add query strings or custom header parameters to your methods. You also might need to update the integration request to [[AWS API Gateway data mapping|map input data]] from the method request to the format required by the backend, or to [[AWS API Gateway error handling|handle errors]] or [[AWS API Gateway input validation|validate input]].

When you deploy your API, you're creating a [[AWS API Gateway stage|stage]]. Each new deployment is added to an existing stage, or creates a new one. This is a way to implement versioning.

## Features

**Reduce latency and throttle traffic**: API Gateway provides end users with the lowest possible latency for API requests and responses by taking advantage of the [[AWS CloudFront]] global network of [[Edge locations|edge locations]]. With this service, you also can **throttle** traffic and authorize API calls to ensure that backend operations withstand traffic spikes and backend systems are not unnecessarily called.

**Build-in, flexible authorization options**: API Gateway gives you several options for authorization. You can authorize access to your APIs with [[IAM]] and [[AWS Cognito]]. If you use [[OAuth token|OAuth tokens]], API Gateway also offers native [[OIDC]] and [[OAuth 2]] support. To support custom authorization requirements, you can invoke a [[AWS Lambda]] authorizer from Lambda. With a **Lambda authorizer**, you can develop your own authorization code using a custom Lambda function.

**API keys for third-party developers**: If you’re using REST APIs, API Gateway helps you manage the ecosystem of third-party developers accessing your APIs. You can create API keys on API Gateway, set fine-grained access permissions on each API key, and distribute them to third-party developers to access your APIs. API keys are not a primary authorization mechanism for your APIs, but provide you the ability to track usage for specific users or services. You can also define usage plans that set throttling and request quota limits for each API key. The use of API keys is optional.

**Monitoring and logs**: There lots of metrics and logs collected by [[AWS CloudWatch]] for API Gateway, see [[AWS API Gateway monitoring]].
