> One implementation of [[AWS API Gateway]]

[[REST]] defines a set of functions such as GET, PUT, and DELETE that clients can use to access server data. Clients and servers exchange data using HTTP. REST API executes compute services on client calls. API Gateway REST APIs use a request-response model, where a client sends a request to a service and the service responds back synchronously. This kind of model is suitable for many different kinds of applications that depend on synchronous communication.

The main feature of a REST API is **statelessness**. Statelessness means that servers do not save client data between requests. Client requests to the server are similar to URLs you type in your browser to visit a website. The response from the server is plain data, without the typical graphical rendering of a web page.

REST APIs offer API proxy functionality and API management features in a single solution. REST APIs also offer API management features such as usage plans, API keys, publishing, and monetizing APIs.

REST API Gateway could be used for existing apps, while [[AWS HTTP API Gateway]] is designed and optimized for a usage with private subnets of Lambda functions.

## Endpoint types

Although it is important to consider your current and future needs when creating your REST API endpoint in API Gateway, it is possible to change the endpoint type. Changing your API endpoint type requires you to update the API's configuration. You can change an existing API type using the API Gateway console, the AWS CLI, or an AWS SDK for API Gateway.

The following endpoint type changes are supported:

1. From edge-optimized to regional or private
2. From regional to edge-optimized or private
3. From private to regional

**You cannot change a private API endpoint into an edge-optimized API endpoint.**

### Regional endpoint

The regional endpoint is designed to reduce latency when calls are made from the same [[Region]] as the API. In this model, API Gateway **does not** deploy its own [[AWS CloudFront]] distribution in front of your API. Instead, traffic destined for your API will be directed straight at the API endpoint in the Region where you’ve deployed it.

This endpoint type gives you *lower latency* for applications that are invoking your API from within the same Region (for example, an API that is going to be accessed from EC2 instances within the same Region). 

The regional endpoint provides you with the flexibility to deploy **your own CloudFront** distribution or [[CDN]] in front of API Gateway and control that distribution using your own settings for customized scenarios. An example of this might be to design for disaster recovery scenarios or implement load balancing in a very customized way.

### Edge-optimized endpoint

The edge-optimized endpoint is designed to help you **reduce client latency** from anywhere on the internet. If you choose an edge-optimized endpoint, API Gateway will **automatically** configure a fully managed [[AWS CloudFront]] distribution to provide lower latency access to your API.

This endpoint-type setup reduces your first hit latency for your API. An additional benefit of using a managed CloudFront distribution is that you don’t have to pay for or manage a CDN separately from API Gateway.

### Private endpoint

The private endpoint is designed to expose APIs only inside your selected [[AWS VPC]]. This endpoint type is still managed by API Gateway, but requests are only routable and can only originate from within a single virtual private cloud that you control.

This endpoint type is designed for applications that have very secure workloads, such as healthcare or financial data that cannot be exposed publicly on the internet. There are no data transfer-out charges for private APIs. However, [[AWS PrivateLink]] charges apply when using private APIs in API Gateway.

## Caching

You can turn on API caching in API Gateway to cache your endpoint's responses. With caching, you can reduce the number of calls made to your endpoint and also improve the latency of requests to your API. This is an optional configuration only available for REST APIs, but definitely an option that you want to consider based on your use cases.

When caching is turned on, API Gateway caches responses from your endpoint for a specified Time-to-Live (TTL) period. API Gateway then responds to a request by looking up the endpoint response from the cache instead of making a request to your endpoint. There are two big benefits of using the cache:

1. It reduces overall latency for serving requests.
2. It minimizes the number of requests that need to be made to your backend.

This becomes even more valuable as you scale and want to reduce the amount of calls to your backend resources.

**Caching is charged at an hourly rate**: Keep in mind that data caching is charged at an hourly rate that is dependent on the cache size you select, regardless of the number of API calls being cached. So be thoughtful in choosing the cache size, and consider the amount of data you intend to cache. Two ways to verify caching:

1. [[AWS CloudWatch|CloudWatch Metrics]]: `CacheHitCount` and `CacheMissCount`.
2. Create a timestamp and include it in your API response.

### Caching per API stage 

Configuration choices for stage caching include the following.

**Provision between 0.5 GB and 237 GB of cache**: When you turn on caching, you can configure the size of the cache anywhere from half a gig to 237 gigabytes, and you can also configure and customize the maximum TTL for each cache entry. 

**Set TTL in seconds**: The default TTL value for API caching is 300 seconds. The maximum TTL value is 3,600 seconds. When you set TTL=0, caching is turned off within API Gateway.

**Turn on encryption of cache data**: You can also encrypt the cached data if you need to. 

**Only GET methods will be cached**: When you turn on caching in a stage's cache settings, only GET methods are cached. We recommend that you don’t cache other types of calls unless you have very specific reasons. 

**Configure per method**: You can override stage-level settings for individual methods. Turn caching on or off for specific methods, increase or decrease the TTL, or turn encryption on or off for cached responses.

You can also use parameters in the method to form cache keys so that API Gateway caches the method's responses depending on the parameter values used.

## Pricing

With API Gateway, you only pay when your APIs are in use. When considering the pricing model for REST APIs, there are two different aspects to consider. Pricing options are set per [[AWS API Gateway stage|stage]].

**Flat Charge**: REST APIs for API Gateway have a flat charge **per million** API Gateway requests. With API Gateway, you only pay when your APIs are in use at a set cost per million requests. The API Gateway free tier includes one million API calls per month for up to 12 months.

**Data transfer out**: An additional cost to factor into your cost estimates is the data transfer out of AWS that will be charged at standard AWS prices. You may incur additional charges if you use API Gateway in conjunction with other AWS services or transfer data out of AWS. Private API endpoints don’t have data transfer-out charges. Instead, [[AWS PrivateLink]] charges apply.

**Optional cache**: if enabled it adds some cost to your.