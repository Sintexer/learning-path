**Event Sourcing is a way of persisting data where, instead of storing the current state of an entity, you store the full sequence of events that led to that state.** The current state is never stored directly as the source of truth — it's always a **derived, computed value**, obtained by replaying the events in order.

Contrast this with what you're used to:

**Traditional ("state-oriented") persistence:**

```sql
UPDATE accounts SET balance = 120 WHERE id = 42;
```

The old value (was it 80? 100?) is gone. The database just holds "current truth," and history is destroyed on every write unless you separately maintain an audit log.

**Event-sourced persistence:**  
You never `UPDATE` a balance field at all. Instead, you append immutable facts:

```
AccountOpened(accountId: 42, initialBalance: 0)
Deposited(accountId: 42, amount: 100)
BetPlaced(accountId: 42, amount: 20)
BetWon(accountId: 42, amount: 40)
```

To get "current balance," you **replay**: 0 + 100 − 20 + 40 = **120**. That computed number might be cached somewhere for performance, but the **events are the actual source of truth** — the cached balance is just a convenience/projection, and could always be thrown away and rebuilt from the event log.

## Event Sourcing Forms

There are two architecturally different setups people mean by "event sourcing," and they have different answers here:

**Setup A — Kafka itself IS the event store** (no separate database for that entity's write-side state). Here, "writing an event" and "publishing an event" are **the same single operation** — you just produce to Kafka. There's no separate "write to DB, then notify Kafka" step to go inconsistent between, because there's only one system involved. The risk shifts to: did the produce itself succeed? You handle that with an **idempotent producer** (`enable.idempotence=true`) so retries-on-timeout don't create duplicates, and you don't tell the calling client "success" until you get a broker acknowledgment (`acks=all`).

**Setup B — a dedicated event store database is the source of truth** (e.g., a Postgres table, or EventStoreDB), and Kafka is a **downstream** propagation mechanism to notify other services / feed CQRS projections. Here you have exactly the failure mode : the DB write (the real, authoritative event) succeeds, but the subsequent "publish to Kafka" step can fail or crash before it happens. **This is precisely the dual-write problem the [[Outbox Pattern]] solves** — write the event to an outbox table in the _same DB transaction_ as the actual event append, so it's atomically safe, then a separate relay process asynchronously drains the outbox into Kafka, retrying until it succeeds. Nothing is ever lost, because the event's existence was never dependent on the Kafka publish succeeding — only on the DB transaction, which is a single-system atomic operation. Full details next topic.

## The natural fit with Kafka

This should click for you immediately given everything we've covered: **[[Kafka]]'s fundamental model — an ordered, immutable, append-only log per partition — is structurally identical to what event sourcing needs.** This isn't a coincidence; many systems literally use Kafka topics (particularly **compacted topics**, which we'll touch on) as their actual event store, rather than a separate specialized "event store" database. You already understand offsets, ordering-per-key, and replay — you now know that's _exactly_ the mechanics event sourcing runs on.

**Event Sourcing naturally produces a stream of events as a side effect of normal operation.** That stream is a _perfect_ input for building [[CQRS]] read models — each read-side projection just subscribes to the event stream and builds its own denormalized view, independently, at its own pace (this is literally [[Kafka Consumer#Consumer groups]]).

So: **Event Sourcing determines how the write side stores its data (as events). CQRS determines that reads and writes use separate models.** They compose beautifully — ES gives you a ready-made stream to feed CQRS read models — which is _why_ they're so often mentioned in the same sentence in system design discussions. But **neither requires the other.** You can have CQRS with a boring state-based [[PostgreSQL]] write side (using CDC to generate the change stream instead of true domain events). You can have Event Sourcing with only one single read model (no CQRS-style multiple specialized views at all) — just replaying events to serve a single API.