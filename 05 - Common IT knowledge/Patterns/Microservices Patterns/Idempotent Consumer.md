> **Central question:** If the broker can deliver a message more than once, how do we prevent duplicate business effects?

From the [[Outbox Pattern]], we know this sequence is unavoidable in a basic implementation:

```
Publish event
Broker accepts it
Publisher crashes before recording success
Publisher retries
```

The consumer may receive the same event *twice*. Rather than assuming duplicates will never happen, we make processing **idempotent**.

This pattern also relies on [[Idempotancy Key]] and [[Version Guard]].

```
Producer local transaction
    Business update + outbox
                |
                v
       At-least-once transport
                |
                v
Consumer local transaction
    Deduplication + business update + optional outbox
                |
                v
          Next service
```

Each service owns a local atomic boundary.

Between services, we rely on:

- Durable messages.
- Retries.
- Idempotency.
- Ordering rules where needed.
- Saga compensation for business failures.
