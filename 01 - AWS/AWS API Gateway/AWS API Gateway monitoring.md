After your APIs are deployed, you can use [[AWS CloudWatch]] Metrics to monitor performance of deployed APIs. API Gateway has seven default metrics out of the box:

1. **Count** - Total number of API requests in a period.
2. **Latency** - Time between when API Gateway receives a request from a client and when it returns a response to the client; this includes the integration latency and other API Gateway overhead.
3. **IntegrationLatency** - Time between when API Gateway relays a request to the backend and when it receives a response from the backend.
4. **4xxError** - Client-side errors captured in a specified period.
5. **5xxError** - Server-side errors captured in a specified period.
6. **CacheHitCount** - Number of requests served from the API cache in a given period.
7. **CacheMissCount** - Number of requests served from the backend in a given period, when API caching is turned on.

In addition, consider the value of turning on detailed metrics so you can see these metrics at the method level. This is where you would be able to see details about gets, posts, deletes, and so on.

**API Gateway overhead**. The difference between  **Latency** and **IntegrationLatency** metrics gives you your API Gateway overhead. Together, these metrics can help you fine-tune your applications and see where the bottlenecks are.

## Logs

API Gateway has two types of [[AWS CloudWatch|CloudWatch]] logs built in: **Execution logging** and **Access logging**.

- **Execution logging** - logs what’s happening on the roundtrip of a request. You can see all the details from when the request was made, the other request parameters, everything that happened between the requests, and what happened when API Gateway returned the results to the client that’s calling the service. Execution logs can be useful to troubleshoot APIs, but **can result in logging sensitive data**. Because of this, it is recommended you don't enable Log full requests/responses data for production APIs. In addition, there is a cost component associated with logging your APIs.
- **Access logging** - provides details about who's invoking your API. This includes everything including IP address, the method used, the user protocol, and the agent that's invoking your API. Access logging is fully customizable using JSON formatting. If you need to, you can publish them to a third-party resource to help you analyze them.

## AWS X-Ray integration

You can use [[AWS X-Ray]] to trace and analyze user requests as they travel through your [[AWS API Gateway]] APIs to the underlying services. With X-Ray, you can understand how your application is performing to identify and troubleshoot the root cause of performance issues and errors. X-Ray gives you an **end-to-end view of an entire request**, so you can analyze **latencies** and errors in your APIs and their backend services. You can also configure sampling rules to tell X-Ray which requests to record, and at what sampling rates, according to criteria that you specify.

## AWS CloudTrail integration

[[AWS CloudTrail]] captures all API calls for API Gateway as events, including calls from the API Gateway console and from code calls to your API Gateway APIs. 

Using the information collected by CloudTrail, you can determine the request that was made to API Gateway, the IP address from which the request was made, who made the request, when it was made, and additional details. You can view the most recent events in the CloudTrail console in Event history.

CloudTrail captures all API calls for API Gateway as events.

- IP address, requester, and time of request are included.
- Event history can be reviewed.
- Create a trail to send events to an [[AWS S3]] bucket.