Writing to two independent systems where you need "both or neither," but have no atomic mechanism spanning both.

You have a service that needs to do **two things** when handling a request:

1. Update its own database (e.g., mark an order as `SHIPPED` in [[PostgreSQL]])
2. Publish an event to [[Kafka]] so other services know it happened (e.g., `OrderShipped` event, so a notifications service can email the customer)

These are **two independent systems** (Postgres and Kafka). You cannot wrap a database [[Transaction]] and a [[Kafka Producer|Kafka produce]] call into one atomic operation — there's no distributed transaction spanning both by default (and [[Two-Phase Commit]] across heterogeneous brokers and databases is slow, fragile, and often unsupported). So you're left with an unavoidable ordering choice, and **both naive orderings are broken**:

**Option 1: Update DB first, then publish to Kafka.**  
What if the process crashes (or the network fails) _after_ the DB commit succeeds but _before_ the Kafka publish happens? The order is marked shipped in your database, but **no event was ever sent** — downstream services never find out. Silent inconsistency.

**Option 2: Publish to Kafka first, then update DB.**  
What if the Kafka publish succeeds, but the subsequent DB update fails or the process crashes right after? Now other services think the order shipped (they got the event), but **your own database says otherwise**. Even worse — an inconsistency that's now been broadcast to the rest of the system.

> [!warning] Failure modes & trade-offs
> The Dual-Write failure window is tiny in happy-path tests (milliseconds between two calls), so it passes QA and only surfaces under deploys, GC pauses, broker leader elections, or OOM kills. Even wrapping both calls inside a Spring `@Transactional` method does **not** fix it — if the Kafka publish succeeds and the DB commit subsequently fails (constraint violation, deadlock, network blip), the event cannot be rolled back.

## Solution

Solution: Don't actually do two writes — do one write, and defer the second.

The trick: instead of writing to your database **and separately** to Kafka, you write to your database **and an "outbox" table, in the same database, in the same transaction** — the [[Outbox Pattern]].

Since both statements are in **one transaction, against one database**, this is a completely normal, fully atomic operation — either both rows change, or neither does. No distributed transaction needed at all, because you never actually touched two different systems in that step. The "event" is just another row in the same database, later relayed by a poller or [[Change Data Capture]] (CDC) to Kafka with [[At-least-once delivery]].

Other valid alternatives when an outbox isn't practical:
- **[[Change Data Capture]] (CDC)** directly on the database [[WAL]] (e.g., Debezium), turning the DB log itself into the single source of truth.
- **[[Event Sourcing]]**, where the event store is the single write target and projections update asynchronously.
- **[[Reconciliation]]** jobs as a safety net when integrating with third-party systems you don't control.

Dual-Write anti-pattern:::Updating two independent systems (e.g., DB and message broker) without an atomic boundary, leaving a crash window where one succeeds and the other fails.

## Why does wrapping a DB write and a Kafka publish in a single `@Transactional` block still fail as a Dual-Write? #card
The broker publish is not enrolled in the database transaction:
- If you publish **after** commit, a crash or network error loses the event.
- If you publish **before** commit, a commit failure (deadlock, constraint error, DB crash) leaves a phantom event in Kafka that cannot be rolled back.
Fix with the [[Outbox Pattern]] or [[Change Data Capture]].

Never treat a database write plus an external broker/cache write as atomic; collapse them into a single local write (Outbox or CDC) or pair with [[Reconciliation]]. #card-reminder

## Related
- [[Outbox Pattern]]
- [[Change Data Capture]]
- [[2PC vs Saga]]
- [[Two-Phase Commit]]
- [[Event Sourcing]]
- [[Reconciliation]]
- [[Delivery Semantics]]
