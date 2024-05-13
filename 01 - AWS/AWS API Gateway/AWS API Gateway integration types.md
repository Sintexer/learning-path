When you choose an integration type, that determines how method request data is passed to the backend. As part of [[AWS API Gateway components#Resource method|creating the method]], you must choose an **integration type**.

Choosing the right integration type for your API Gateway method determines how request data is passed to the backend. Here's a quick overview of the available options:

- **Lambda function integration:** Ideal for integrating with [[AWS Lambda function|Lambda functions]]. [[AWS API Gateway]] acts as a *proxy*, forwarding requests to the function and providing event details in the handler.
- **HTTP Endpoint:** Suitable for public web applications, allowing clients to interact directly with the endpoint. Configure both request and response mappings for this type.
- **AWS Service:** Exposes AWS service actions directly through the API. For example, send messages to an [[AWS SQS]] queue.
- **Mock:** Returns a pre-defined response without sending the request further. Useful for *health checks* or hardcoded responses.
- **VPC Link:** Connects to a Network Load Balancer in your private [[AWS VPC]], enabling access to resources not publicly accessible.

The choice depends on your specific needs and backend architecture. Consider factors like security, performance, and flexibility when selecting the appropriate integration type for your API Gateway method.

## Lambda function integration

When you are using [[AWS API Gateway]] as the gateway to a [[AWS Lambda function]], you’ll use the Lambda integration. This will result in requests being proxied to Lambda with request details available to your function handler in the event parameter, supporting a streamlined integration setup. The setup can evolve with the backend without requiring you to tear down the existing setup. For integrations with Lambda functions, you will need to set an [[IAM role]] with required permissions for API Gateway to call the backend on your behalf.

## HTTP Endpoint

HTTP integration endpoints are useful for public web applications where you want clients to interact with the endpoint. This type of integration lets an API expose HTTP endpoints in the backend.

When the proxy is not configured, you’ll need to configure both the integration request and the integration response, and set up necessary data mappings between the method request-response and the integration request-response.

If the proxy option is used, you don’t set the integration request or the integration response. API Gateway passes the incoming request from the client to the HTTP endpoint and passes the outgoing response from the HTTP endpoint to the client.

## AWS Service

**AWS Service** is an integration type that lets an API expose AWS service actions. For example, you might drop a message directly into an [[Amazon SQS]] queue.

## Mock

**Mock** lets API Gateway return a response without sending the request further to the backend. This is a good idea for a health check endpoint to test your API. Anytime you want a **hardcoded response** to your API call, use a Mock integration.

## VPC Link

With [[AWS VPC]] Link, you can connect to a [[AWS NLB]] to get something in your private VPC. For example, consider an endpoint on your [[EC2]] instance that’s not public. API Gateway can’t access it unless you use the VPC link and you have to have a Network Load Balancer on your backend. For an API developer, a **VPC Link** is functionally equivalent to an integration endpoint.