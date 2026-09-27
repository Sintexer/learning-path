
**Idempotency means repeating the same logical operation does not create an additional business effect.**

It does not necessarily mean:

- The code executes only once.
- The network request is delivered only once.
- Every retry returns an identical HTTP response.

The important question is:

> **Which operation, at which boundary, must not have its effect repeated?**

That explains why we talk about client-request, producer, consumer, and external-operation idempotency.

Consider checkout:

```text
Client
   |
   | POST /orders
   v
Order Service
   |
   | Publish OrderCreated
   v
Message Broker
   |
   | Deliver OrderCreated
   v
Payment Service
   |
   | POST /charges
   v
Payment Provider
```

Every arrow has an uncertain-outcome problem:

```text
Receiver succeeds
Response / acknowledgment is lost
Sender retries
```

Protecting one arrow does not automatically protect the others.

| Boundary | Protection |
|---|---|
| Client → Order Service | Request idempotency |
| Producer → Broker | Broker-supported producer idempotency, where available |
| Broker → Consumer database | Consumer idempotency |
| Payment Service → Provider | External-operation idempotency |

An [[Idempotancy Key]] tells the receiver: “This request is another attempt at the same operation, not a new operation.” [[API Idempotency]] is another form of idemppotency, which guarantees client-server idempotency.

## Broker-level producer idempotency

For example, [[Kafka]]’s idempotent producer protects against certain duplicate records caused by **producer transmission retries**.

```text
Producer sends a record
Broker stores it
Acknowledgment is lost
Producer retries the transmission
```

Kafka uses producer identity and sequence information to recognize these retry duplicates within its supported guarantees.

It does not deduplicate arbitrary application messages by your `eventId`.

Suppose an outbox publisher:

```text
Publishes E123 successfully
Crashes before marking E123 published
Restarts and publishes E123 again
```

The second publication can be a new application-level send. Broker producer idempotency alone does not generally recognize it as the same business event.

Both records can contain:

```json
{
  "eventId": "E123"
}
```

and both can still reach consumers.

> Broker-level producer idempotency protects transport retries—not every possible application replay.

## Application-level production idempotency

This means the application avoids creating a second logical event for the same operation.

```text
Client retries checkout
    ↓
Request idempotency returns existing order
    ↓
No second OrderCreated event is created
```

A transactional outbox helps ensure:

```text
Order creation + its event record commit together
```

But the outbox does **not** independently determine whether two requests are the same checkout. That requires request identity and business constraints.

## Consumer Idempotency: Broker → Business Effect

> A consumer is idempotent when receiving the same message again does not repeat its business effect.

```text
Consumer receives E123
Consumer reserves inventory
Database commits
Consumer crashes before acknowledgment
Broker redelivers E123
```

The second delivery must not reserve inventory again.

## The standard database technique

Create a deduplication table:

```sql
CREATE TABLE processed_event (
    consumer_name VARCHAR(100) NOT NULL,
    event_id UUID NOT NULL,
    PRIMARY KEY (consumer_name, event_id)
);
```

Process the message:

```text
BEGIN

Insert (consumer_name, event_id)
    ON CONFLICT DO NOTHING

If insertion succeeded:
    Apply business changes
    Insert any outgoing events into outbox

COMMIT

Acknowledge broker
```

**The deduplication marker and business change must share one transaction.**

Otherwise:

- Marker commits but effect fails → work is incorrectly skipped.
- Effect commits but marker fails → work may be repeated.

## Why include consumer name?

The same event can legitimately affect several handlers:

```text
Inventory handler: reserve stock
Analytics handler: update statistics
```

Each needs its own processing identity.

Use a stable **logical handler name**, not a unique name per worker instance.

## Business-Operation Idempotency: Stronger Than Message Deduplication

Now suppose two different messages describe the same action:

```text
Message E1: reserve stock for operation R123
Message E2: reserve stock for operation R123
```

Event-ID deduplication accepts both because:

```text
E1 ≠ E2
```

But the business operation is the same:

```text
R123 = R123
```

A business uniqueness constraint protects that:

```sql
UNIQUE (reservation_operation_id)
```

This is why message IDs and operation IDs are not interchangeable.

| Identifier | Meaning |
|---|---|
| `event_id` / `message_id` | Which message is this? |
| `operation_id` | Which intended business action is this? |
| `order_id` | Which business entity does it concern? |
| `trace_id` | Which diagnostic execution does it belong to? |

### Why not always use the order ID?

One order can legitimately have:

- Multiple payment attempts.
- A capture and a refund.
- Several partial refunds.
- Multiple shipments.

An entity ID identifies the subject, not necessarily a unique action.

A key should identify the **specific intended operation**.

## External-Operation Idempotency

This protects calls such as:

```text
Payment Service → Payment Provider
```

Use a stable payment operation ID across retries.

```text
Attempt 1: operation P123 → timeout
Attempt 2: operation P123 → existing result
```

Do not generate a new ID for every attempt.

### Why local consumer deduplication is insufficient

Consider:

```text
Insert processed-event marker
Call provider
Provider charges
Database transaction rolls back
```

The provider’s charge survives the rollback.

The message is redelivered, and your database sees no marker.

The safe design is:

```text
Local transaction:
    Deduplicate incoming message
    Create durable payment operation
    Commit

Worker:
    Execute payment using stable provider key
    Reconcile uncertain outcomes
```

The provider must support safe duplicate submission or an equivalent recovery protocol.

If it supports neither idempotency nor reliable outcome lookup, automatic exactly-once external effects generally cannot be guaranteed.

## Naturally Idempotent Operations

Some operations are inherently repeatable:

```sql
UPDATE customer_preferences
SET marketing_enabled = false
WHERE customer_id = :id;
```

Repeating the assignment does not repeatedly decrement or increment anything.

Contrast:

```sql
UPDATE wallet
SET balance = balance - 50
WHERE id = :id;
```

Repeating it charges twice.

### But natural idempotency has limits

Suppose:

```text
Event 1: set marketing_enabled = false
Event 2: set marketing_enabled = true
Late duplicate of Event 1 arrives
```

The duplicate now overwrites newer intent.

So an operation can be idempotent in isolation but still unsafe when reordered.

Use versions or conditional transitions where needed.

Also, associated effects can be non-idempotent:

```text
Set status = CANCELLED
Send cancellation email
```

The assignment may be repeatable; the email may not be.

## How the Layers Work Together

For checkout, the identifiers might be:

```text
Checkout request key: checkout-123
Order ID:             O456
OrderCreated event:   E789
Payment operation:    P321
Refund operation:     R654
```

Their responsibilities:

```text
checkout-123
    Prevent duplicate order creation.

E789
    Prevent the same handler from applying this event twice.

P321
    Prevent repeated attempts from creating another charge.

R654
    Prevent repeated cleanup attempts from creating another refund.
```

Broker-supported producer idempotency can additionally suppress transmission duplicates, but it does not replace these business identities.

**Do not force one identifier to serve every purpose.** Propagate causal relationships, but keep distinct operations distinct.

## Practical Rules

### A. Retries must reuse the key

```text
Same operation → same key
New operation  → new key
```

A new random key on every retry defeats idempotency.

### B. Scope keys appropriately

Keys may need to be scoped by customer, tenant, account, or endpoint.

Otherwise, unrelated callers can conflict—or accidentally access another caller’s result.

### C. Same key, different payload is an error

A retry should not silently change:

```text
Charge $50 → Charge $500
```

Store or compare relevant request parameters.

### D. Retention limits the guarantee

If a key expires after 24 hours, a retry after 48 hours may be treated as new.

Deduplication retention must align with replay, retry, and disaster-recovery windows.

### E. Redis is not automatically an atomic deduplication solution

This is unsafe without additional coordination:

```text
Mark event processed in Redis
Update business database
```

Those are separate writes. For database effects, a deduplication record in the same transactional database is usually simpler.

### F. Idempotency is not ordering

- Idempotency prevents repeated effects.
- Ordering protects the sequence of different effects.
- Concurrency control prevents conflicting simultaneous updates.
- Reconciliation discovers and repairs uncertain or inconsistent outcomes.

These mechanisms complement one another.