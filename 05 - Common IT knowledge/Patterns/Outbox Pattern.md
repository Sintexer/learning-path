
The **Outbox pattern** enables us to reliably update a database and publish a message when they are two separate systems. This pattern solves the [[Dual-Write]] problem.

## The scenario

Your Order Service needs to:

1. Save an order in PostgreSQL.
2. Publish `OrderCreated` to a broker such as Kafka or RabbitMQ.

Inventory Service consumes that event and reserves stock.

```text
Order Service
    ├── PostgreSQL: save order
    └── Message broker: publish OrderCreated
```

These are **two writes to different systems**, hence “dual write.” A normal Spring database transaction does not automatically cover the broker.

### Attempt A: commit the database, then publish

```java
Order order = orderService.saveInTransaction(request);

// The database transaction has committed.
messagePublisher.publish(new OrderCreated(order.getId()));
```

Possible sequence:
1. Order is committed to PostgreSQL.
2. Application crashes.
3. Event is never published.

Result:
Database: order exists. Inventory Service: never learns about it. Restarting the application does not automatically tell it which events were never sent.

### Attempt B: publish first, then commit the database

```text
1. Publish OrderCreated.
2. Try to save the order.
3. Database transaction fails.
```

Now Inventory Service may reserve stock for an order that does not exist.

Changing the order of the writes only changes **which inconsistency you can get**.

### Attempt C: put both calls inside `@Transactional`

```java
@Transactional
public void createOrder(CreateOrderRequest request) {
    Order order = orderRepository.save(...);
    messagePublisher.publish(new OrderCreated(order.getId()));
}
```

This looks atomic, but normally it is not.

With a database transaction manager:

- The database write participates in the transaction.
- The broker publish does not automatically participate.

The broker might accept the event, then the database commit might fail.

> `@Transactional` does not turn arbitrary network calls into one atomic transaction.

Similarly, an in-memory “publish after commit” callback avoids publishing before commit, but still leaves a crash window after the database commits.

## Implementation

Instead of writing to the database and broker directly, write **both the business data and the intent to publish** into the same database transaction.

```text
One PostgreSQL transaction:
    Insert order
    Insert OrderCreated into outbox
    Commit
```

Then a separate publisher delivers outbox rows to the broker.

```text
Order Service
    |
    | One local transaction
    v
PostgreSQL
    ├── orders
    └── outbox
             |
             | Publisher reads committed rows
             v
        Message broker
             |
             v
       Inventory Service
```

The outbox is simply a database table containing messages waiting to be delivered.

### Why does this help?

A local database transaction gives us:

```text
Order saved AND event intent saved
```

or:

```text
Neither saved
```

There is no committed order without its corresponding durable event intent, assuming all order-creation paths use this transaction.

We have not made the database and broker atomic. We have made the **obligation to publish durable**.

## Basic schema

For PostgreSQL:

```sql
CREATE TABLE outbox_event (
    id             UUID PRIMARY KEY,
    aggregate_id   UUID NOT NULL,
    event_type     VARCHAR(100) NOT NULL,
    payload        JSONB NOT NULL,
    created_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    published_at   TIMESTAMPTZ
);
```

For multiple workers, you can claim outbox rows using the **same short-transaction lease mechanism** we covered for Saga workers:

```text
Transaction 1: claim an outbox row
Outside transaction: publish and wait for acknowledgment
Transaction 2: mark published, checking claim ownership
```

The minimal table above omits those lease columns to keep the core idea visible.

## Crash recovery: examine every boundary

This is the most important part.

## Case A: crash before the order transaction commits

The transaction rolls back. No partial result.

## Case B: crash after database commit, before publication

```text
Order committed
Outbox event committed
CRASH
```

The event remains in the outbox with:

```text
published_at = NULL
```

A publisher later discovers and sends it.

If a publisher had already claimed the row, another worker can reclaim it after its lease expires.

## Case C: broker accepts the event, but acknowledgment is lost

```text
Publisher → sends event E1
Broker    → accepts E1
Network   → loses acknowledgment
Publisher → sees timeout
```

The publisher cannot know whether delivery succeeded.

It retries **the same event ID, E1**.

The broker may therefore contain two deliveries of E1.

## Case D: acknowledgment arrives, then publisher crashes

```text
Broker accepts E1
Publisher receives acknowledgment
CRASH before published_at is saved
```

The database still says:

```text
published_at = NULL
```

After recovery, the publisher sends E1 again.

Again: duplicates are possible.

## Case E: publication is acknowledged and recorded

```text
Broker accepts E1
Publisher receives acknowledgment
Publisher commits published_at
```

Normal polling no longer selects that row.

Broker durability still depends on appropriate acknowledgment and replication settings.

# 10. Connecting Outbox back to Saga orchestration

Related to [[Saga pattern]].

A message-based coordinator often needs to:

```text
Save Saga state: PAYMENT_PENDING
Send command: ChargePayment
```

That is another [[Dual-Write]] problem.

Use:

```text
One database transaction:
    Update Saga to PAYMENT_PENDING
    Insert ChargePayment command into outbox
```

If the coordinator crashes after commit, the command still gets delivered.

Outbox can carry **commands or events**. It is not restricted to event-driven choreography.

## Publishing

Why Two Approaches Exist? **Polling** is attractive because almost every team already knows how to:

- Query a database.
- Run a scheduled Spring worker.
- Publish a message.

But as workloads grow, polling introduces database queries, bookkeeping updates, and delivery latency.

CDC - **Change Data Capture** - uses the database’s existing change stream instead. It can reduce polling overhead and deliver changes quickly, but adds specialized infrastructure and operational responsibilities.

Both approaches require the [[Idempotancy Key]] and idempotent consumers to be implemented.

Neither is universally better.

|Question|Polling|CDC|
|---|---|---|
|How are events discovered?|Query outbox rows|Read committed changes from transaction log|
|Who usually operates it?|Application team|Application and/or data-platform team|
|Where is progress tracked?|Outbox status and worker state|Connector offsets and database log position|
|Typical starting complexity|Lower|Higher unless CDC already exists|
|Main operational pressure|Queries, updates, table maintenance|Log retention, connector health, recovery|

### Polling Publishing

> **Polling publisher:** periodically query the outbox table and publish pending rows.

The publisher may run:

- Inside the Order Service application.
- As a separate application using the same outbox database.

A separate deployment lets you scale and restart publishing independently, but also introduces another component to operate.

Why implement polling first? Because it has a relatively small conceptual footprint:

> “We already have a database and a Spring application. Add a table and a worker.”

That is often the right trade-off for moderate event volumes.

Polling frequency is another limitation. With an interval of 5 seconds you have worst case of event is delayed for 5 seconds. It also introduces the write amplification.

### CDC Publishing

Change Data Capture. 

> **Transaction-log tailing / CDC:** read committed database changes from the database’s transaction log and publish them.

A database maintains a durable record of changes for recovery and replication - transaction log ([[WAL]] in PostgreSQL, binlog in MySQL)

With polling, the application typically writes:

```
published_at = ...
```

With CDC, progress is usually tracked through connector offsets and the source log position.

Conceptually:

```
Connector has processed changes through position X.
```

The connector does not normally update every outbox row to mark it delivered. That eliminates some polling-related bookkeeping writes.

However:

> “The connector read the event” and “every downstream consumer processed the event” are different milestones.

This distinction also applies to polling: a broker acknowledgment is not a consumer acknowledgment.

CDC moves work out of the application publisher, but it creates a relationship between:
- Connector health.
- Database log retention.
- Database disk capacity.

