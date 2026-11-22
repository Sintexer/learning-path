**Purpose:** Follow one operation across service and messaging boundaries (see also [[Microservices Tracing]]).

```
Gateway span
   └── Order API span
         ├── Database span
         └── Payment HTTP span
```

Key terms:

- **Trace:** related operations forming one execution story.
- **Span:** one timed operation, such as an [[HTTP]] call, [[Kafka]] consume, or database query.
- **Trace context:** identifiers (`trace_id`, `span_id`, sampling flags) propagated between components.

A common stack is **[[OpenTelemetry]]** instrumentation exporting to a tracing backend (e.g., Jaeger, Tempo, Datadog).

### Practical implementation

Propagate trace context through:
- HTTP headers, commonly W3C `traceparent` (and `tracestate`).
- Message headers for asynchronous processing ([[Kafka Message]] headers).

Include relevant identifiers in structured logs so logs, metrics, and traces correlate:

```
trace_id
span_id
order_id
saga_id
event_id
```

### Practical limitations

- **Sampling** means not every trace is retained (head-based sampling decides at entry and can drop rare errors; **tail-based sampling** buffers traces briefly so 100% of errors and high-latency outliers are kept, at higher collector memory cost).
- Missing context propagation across thread pools, async callbacks, or message queues breaks the trace chain.
- Long-running [[Saga|Sagas]] may be better represented by **span links** (linked traces) than one enormous trace.
- Sensitive payloads (PII, tokens, secrets) should not be copied indiscriminately into span attributes.

A trace ID is **not** an [[Idempotency Key|idempotency key]] or durable workflow identifier.

> [!tip] Production reality
> Always attach a **business correlation ID** (`order_id`, `saga_id`) alongside `trace_id` in logs and events. Traces are sampled and expire in days, whereas business correlation IDs let you reconstruct an incident weeks later across logs, DB tables, and [[DLQ]] records.

Distributed Tracing:::Tracking a single logical request across multiple services and async boundaries using a propagated Trace Context (`trace_id`, `span_id`) composed of nested Spans.

## Head-based vs. Tail-based sampling in Distributed Tracing — what is the trade-off? #card
- **Head-based sampling** decides at request entry (e.g., keep 1%): cheap and stateless, but drops 99% of rare errors and tail-latency spikes.
- **Tail-based sampling** buffers spans at the collector until the trace completes: keeps 100% of errors and slow traces, at the cost of higher collector memory and trace-affinity routing.

Never reuse a `trace_id` as an ==Idempotency Key== or durable workflow ID — traces are sampled, ephemeral, and regenerate on client retries. #card-reminder

## Related
- [[Microservices Tracing]]
- [[OpenTelemetry]]
- [[Microservices Service mesh pattern]]
- [[API Gateway]]
- [[Saga]]
- [[Kafka Message]]
