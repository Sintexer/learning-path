> One implementation of [[AWS API Gateway]]

[[HTTP]] APIs, you can create [[RESTful]] APIs with lower latency and lower cost than [[AWS REST API Gateway|REST APIs]]. You can use HTTP APIs to send requests to Lambda functions or to any routable HTTP endpoint.

**HTTP APIs** are optimized for building APIs that proxy to [[AWS Lambda function|Lambda functions]] or HTTP backends, making them ideal for serverless workloads. They do not currently offer API management functionality.

For example, you can create an HTTP API that integrates with a Lambda function on the backend. When a client calls your API, API Gateway sends the request to the Lambda function and returns the function's response to the client.