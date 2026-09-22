**Purpose:** Follow one operation across service and messaging boundaries.

```
Gateway span
   └── Order API span
         ├── Database span
         └── Payment HTTP span
```

Key terms:

- **Trace:** related operations forming one execution story.
- **Span:** one operation, such as an HTTP call or database query.
- **Trace context:** identifiers propagated between components.

A common stack is **OpenTelemetry** instrumentation exporting to a tracing backend.

### Practical implementation

Propagate trace context through:

- HTTP headers, commonly W3C `traceparent`.
- Message headers for asynchronous processing.

Include relevant identifiers in structured logs:

```
trace_id
span_id
order_id
saga_id
event_id
```

### Practical limitations

- Sampling means not every trace is retained.
- Missing context propagation breaks the chain.
- Long-running Sagas may be better represented by linked traces than one enormous trace.
- Sensitive payloads should not be copied indiscriminately into span attributes.

A trace ID is **not** an idempotency key or durable workflow identifier.