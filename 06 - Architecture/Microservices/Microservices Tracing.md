Distributed tracing tracks and visualizes the end-to-end flow of a single request across multiple [[Microservices architecture|microservices]], databases, and async queues. It is essential for debugging latency bottlenecks, cascading failures, and fan-out bugs in distributed systems (see also [[Distributed Tracing]]).

## Key Concepts

- **Trace**: Represents the full end-to-end journey of a request as it travels through services (identified by a global `trace_id`).
- **Span**: Represents a single timed unit of work inside a service (e.g., an [[HTTP]] handler, [[gRPC]] call, SQL query, or [[Kafka Consumer]] poll), recording `start_time`, `duration`, `parent_span_id`, status, and structured attributes/tags.
- **Context Propagation**: Passing trace metadata across process boundaries — via W3C `traceparent` / `tracestate` headers for HTTP/gRPC, and message headers for [[Kafka Message|Kafka]] / RabbitMQ — so child spans link back to the same trace tree.
- **Span Links**: Connect loosely coupled traces (e.g., batch consumers or long-running [[Saga|Sagas]]) without creating an unreadable 10,000-span mega-trace.

## Standard Stack ([[OpenTelemetry]])

Modern systems standardize on **[[OpenTelemetry]] (OTel)** for vendor-neutral instrumentation:
1. **SDK / Auto-instrumentation** in each service creates spans and propagates W3C headers.
2. **OTel Collector** receives spans, scrubs PII, applies sampling, and exports to a backend (Jaeger, Grafana Tempo, Zipkin, Datadog).
3. **Log & Metric Correlation**: Inject `trace_id` and `span_id` into structured JSON logs (and attach business keys like `order_id`, `saga_id`) so engineers can jump directly from a slow trace to exact log lines.

## Sampling Strategies

- **Head-based sampling**: Decides keep/drop at the entry point (e.g., [[API Gateway]]) — cheap, but misses rare errors and p99.9 outliers.
- **Tail-based sampling**: Buffers completed traces in the Collector and retains 100% of error traces and high-latency outliers while down-sampling fast `200 OK` traffic.

> [!warning] Failure modes & trade-offs
> - **Broken context across threads / async queues**: Submitting work to a Java `ExecutorService` or reactive pipeline without context propagation drops `trace_id` and splits one request into orphaned traces.
> - **High-cardinality attribute explosion**: Putting raw SQL strings, unbounded user IDs, or full request bodies into indexed tags/metrics blows up backend storage and leaks PII.
> - **[[Service Mesh]] misconception**: A service mesh proxy (Envoy) creates ingress/egress spans automatically, **but cannot link inbound to outbound calls inside your app** unless your app forwards the `traceparent` header.

## Why can a Trace break into disconnected fragments inside a Java microservice even when OpenTelemetry is installed? #card
Trace context is typically stored in a `ThreadLocal`. When work crosses an async boundary — a custom `ExecutorService`, reactive operator, or [[Kafka]] message without header propagation — the context is lost unless explicitly captured and restored (e.g., via OTel `Context.wrap()` or message header injection).

Correlate traces with logs by injecting ==`trace_id` and `span_id`== into structured logs alongside durable business IDs like ==`order_id` or `saga_id`==. #card-reminder

## Related
- [[Distributed Tracing]]
- [[OpenTelemetry]]
- [[Microservices Service mesh pattern]]
- [[Microservices Sidecar pattern]]
- [[API Gateway]]
- [[Kafka Message]]
- [[Saga]]
