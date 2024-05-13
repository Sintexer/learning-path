API Gateway helps you manage the volume of API calls that are processed through your API endpoint. You can set **throttle** and **quota limits** on your API consumers. This can useful for things such as preventing one consumer from using all of your backend system’s capacity or to ensure that your downstream systems can manage the number of requests you send through.

## API keys

With API Gateway, you can create and distribute API keys to your customers, which can be used to identify the consumer and apply desired usage and throttle limits to their requests. Customers include the API key through `x-API-key` header in requests.

## Usage plans

You can use API keys with usage plans to set up some very specific plans that make sense for your use case. For example, you can perform API key throttling based on a rate and a burst per API key. API key usage can also be used to meter daily, weekly, and monthly usage. 

You can set throttle and quota limits based on API keys through the usage plans feature. You can set up usage plans for:

- **API Key Throttling** per second and burst
- **API Key Quota** by day, week, or month
- **API Key Usage** by daily usage records

## Throttling settings hierarchy

The type and level of throttling applied to a request is dependent on all of the limits involved and are applied in this order:

1. **Per-client**, **per-method** throttling limits that you set for an API stage in a usage plan
2. **Per-client** throttling limits that you set in a usage plan
3. **Default per-method limits** and **individual per-method limits** that you set in API stage settings
4. The **account level limit**

## Token bucket algorithm

- Burst: Maximum size of bucket
- Rate: Number of tokens (requests) added to bucket

The method by which the limits are measured and throttled is based on the token bucket algorithm, which is a widely used algorithm for checking that network traffic conforms to set limits. A token, in this case, counts as a request and the burst is the maximum bucket size.

Requests that come into the bucket are fulfilled at a steady rate. If the rate at which the bucket is being filled causes the bucket to fill up and exceed the burst value, a 429 **Too Many Requests** error would be returned.

API Gateway sets a limit on a steady-state rate and a burst of request submissions per account and per Region. At the account level, by default, API Gateway limits the steady-state request rate to 10,000 requests per second. It limits the burst to 5,000 requests across all APIs within an AWS account. However, as discussed earlier, you can use usage plans to manage limits at a more granular level.