A **reverse proxy** is a server that accepts a request from the client, forwards the request to backend web servers, and returns the results to the client as if the proxy server had processed the request (e.g., NGINX, Envoy, HAProxy). Unlike a [[Forward proxy]], it acts on behalf of the **servers**.

A reverse proxy is good for:

1. **Protecting servers** (hiding backend topology/IPs, WAF filtering, [[Rate Limiting]])
2. **[[Load Balancing]]** (L7 routing across healthy origin instances)
3. **Caching static and semi-static content** (reducing origin load, complementing a [[CDN]])
4. **[[TLS]] termination** (centralizing certificate management and offloading cryptographic work)
5. **Compression & protocol translation** (e.g., serving [[HTTP2]]/[[HTTP3]] to clients while speaking [[HTTP1.1]] or [[gRPC]] to backends)

In microservices, an [[API Gateway]] is a specialized L7 reverse proxy that adds [[Authentication]], [[Authorization]], request routing, and observability.

> [!warning] Failure modes & trade-offs
> - **Lost client IP**: backends see the proxy's IP unless the proxy sets `X-Forwarded-For` / `Forwarded` headers (or uses PROXY protocol), and backends must only trust those headers from known proxies to prevent IP spoofing.
> - **Buffer/timeout mismatches**: if the reverse proxy's idle keep-alive timeout is *longer* than the backend's keep-alive timeout, the proxy can reuse a half-closed socket and trigger sporadic `502 Bad Gateway` errors. Always set backend keep-alive timeout slightly higher than the proxy's.
> - **Single point of failure**: deploy reverse proxies redundantly behind L4 [[Load Balancing]] or anycast DNS.

## Flashcards

Reverse proxy:::An intermediary sitting in front of backend servers that accepts client requests, handles [[TLS]] termination, caching, or [[Load Balancing]], and forwards requests to origins.

Why can mismatched keep-alive timeouts between a reverse proxy and an origin server cause intermittent `502 Bad Gateway` errors?::If the origin closes an idle connection right as the proxy reuses it, the proxy hits a closed socket; set the origin's keep-alive timeout ==longer== than the proxy's.

### How does an [[API Gateway]] differ from a general-purpose reverse proxy? #card

An [[API Gateway]] is a specialized L7 reverse proxy focused on API-level concerns: [[Authentication]]/[[Authorization]] (e.g., [[JWT]] validation), per-tenant [[Rate Limiting]], request/response transformation, versioning, and [[Distributed Tracing]] injection.

## Related
- [[Forward proxy]]
- [[Load Balancing]]
- [[API Gateway]]
- [[CDN]]
- [[TLS]]
- [[Rate Limiting]]
