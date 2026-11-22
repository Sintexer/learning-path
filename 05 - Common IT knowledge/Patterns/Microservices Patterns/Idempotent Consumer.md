> **Central question:** If the broker can deliver a message more than once ([[At-least-once delivery]]), how do we prevent duplicate business effects?

From the [[Outbox Pattern]], we know this sequence is unavoidable in a basic implementation:

```
Publish event
Broker accepts it
Publisher crashes before recording success
Publisher retries
```

The [[Kafka Consumer|consumer]] may receive the same event *twice*. Rather than assuming duplicates will never happen, we make processing **idempotent** (turning [[At-least-once delivery]] into an effectively [[Exactly-once delivery|exactly-once effect]]).

This pattern also relies on [[Idempotency Key]] and [[Version Guard]].

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

Each service owns a local atomic boundary ([[Database Per Service]]).

Between services, we rely on:
- Durable messages.
- [[Retry Pattern|Retries]].
- [[Idempotency]].
- Ordering rules where needed ([[Version Guard]]).
- [[Saga]] compensation for business failures.

## Typical Implementation (Inbox / Processed-Events Table)

Inside **one local database [[Transaction]]**:
1. `INSERT INTO processed_events (event_id, processed_at) VALUES (:id, now()) ON CONFLICT DO NOTHING`.
2. If `0` rows were inserted, this event was already applied → commit/skip cleanly and advance the offset.
3. Otherwise, apply the business state change (and optionally insert downstream events into the local outbox) and commit.

> [!warning] Failure modes & trade-offs
> - **Deduplication outside the DB transaction**: Checking [[Redis]] first and then writing to [[PostgreSQL]] breaks under crashes ([[Dual-Write]]) and races ([[Check-Then-Act]]). Keep the deduplication record in the **same transactional store** as the business state.
> - **Unbounded `processed_events` table**: Without partitioning by time and pruning old rows (past your broker retention + [[DLQ]] replay horizon), the inbox table grows forever and degrades insert latency.
> - **External side effects (email, third-party APIs)**: A local DB transaction cannot wrap an external HTTP call; pair the inbox with an outbound [[Idempotency Key]] supported by the downstream provider and a [[Reconciliation]] job.

Idempotent Consumer:::A consumer that tracks processed message IDs (inbox table) or enforces monotonic state guards inside the same local transaction as its business update, making duplicate deliveries safe.

## How do you implement an Idempotent Consumer atomically in PostgreSQL? #card
In **one local transaction**: insert `event_id` into a `processed_events` table with a `PRIMARY KEY` / `UNIQUE` constraint (`INSERT ... ON CONFLICT DO NOTHING`). If the insert affects 0 rows, skip processing as a duplicate; otherwise apply the business update and commit both together.

Never store an Idempotent Consumer's deduplication state in ==a separate store (like Redis)== from its business database; crashes between the two reintroduce ==Dual-Write duplicates==. #card-reminder

## Related
- [[Delivery Semantics]]
- [[At-least-once delivery]]
- [[Exactly-once delivery]]
- [[Outbox Pattern]]
- [[Idempotency Key]]
- [[Version Guard]]
- [[Dual-Write]]
- [[Check-Then-Act]]
- [[Saga]]
