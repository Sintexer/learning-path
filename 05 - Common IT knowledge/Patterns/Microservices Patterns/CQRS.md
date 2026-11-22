Command Query Responsibility Segregation. **CQRS = using separate models for writing data (commands) versus reading data (queries).**

That's it at its core — deceptively simple. But "separate models" can mean different things depending on how far you take it:

- At minimum: separate _code paths/classes_ for handling writes vs reads, even against the same database
- Taken further (much more common in real distributed systems): **separate data stores entirely** — a write-optimized store ([[OLTP]]) and a completely different, read-optimized store (often [[OLAP]] or denormalized search/cache), kept in sync

### Why would you ever want this? The core motivation

Writes and reads have fundamentally different needs.

**The write side (commands) needs:**
- Strong consistency and validation ("can this bet actually be placed given the account balance and business rules?")
- Transactional integrity ([[ACID]] — money can't disappear or duplicate)
- A normalized structure that avoids data anomalies (classic [[Relational Database|relational design]] — foreign keys, constraints)

**The read side (queries) needs:**
- Speed, and lots of it — reads massively outnumber writes in most systems (think: how many times a user's bet history or live odds are _viewed_ vs how many times a bet is actually _placed_)
- Flexible, often complex querying shapes (aggregations, joins across many entities, filtering, sorting) that would be slow or awkward against a normalized transactional schema
- Data shaped exactly like the UI/API needs it — **denormalized**, precomputed, flattened

If you try to serve both needs from **one single model**, you inevitably compromise both sides. A schema normalized enough to be safe for transactional writes is usually painful and slow to query for rich analytical reads. A schema denormalized enough for fast flexible reads is dangerous to write to directly (data duplication, no strong guarantees). CQRS says: **stop compromising — let each side be optimized independently for what it actually needs.**

### What this looks like architecturally

```
Write path:                                   Read path:
 Client → Command → [Command Handler]         Client → Query → [Query Handler]
                            ↓                                        ↑
                   Write DB (Postgres,                  Read DB (ClickHouse,
               normalized, transactional)               denormalized, fast)
                            ↓
            (sync mechanism — events, CDC, etc.)
                            ↓
                  Updates the read model
```

The write side handles a command, validates it, applies it to its own store. Then, **some mechanism** propagates that change into the read-optimized store, so queries stay up to date (usually with some small delay — this introduces **[[Eventual Consistency|eventual consistency]]** between write and read sides, which is a tradeoff you should be ready to name explicitly).

## CQRS does not require Event Sourcing

You can implement CQRS in a very simple way:

- Write side: a service writes directly to [[PostgreSQL|Postgres]], normally, with regular `UPDATE`/`INSERT` statements — no event log, no special pattern, just a normal transactional write
- Sync mechanism: a background job periodically queries what changed and pushes it into the read store, **or** you use **[[Change Data Capture]] (CDC)** — a tool (like Debezium) that watches the Postgres [[WAL|write-ahead log]] directly and streams row-level changes out, without the application even being aware (or use the [[Outbox Pattern]])
- Read side: ClickHouse (or Elasticsearch, or a denormalized read replica, etc.) gets updated from that stream

**No [[Event Sourcing|event sourcing]] anywhere in this picture.** The write side never stores "the sequence of things that happened" — it just stores current state, like any normal database, and updates get propagated forward. This is a completely valid, very common, simpler form of CQRS.

[[Event Sourcing]] is a _different, additional_ decision about **how the write side itself stores its data** — and it happens to compose beautifully with CQRS, which is why they're so often mentioned in the same breath. But one absolutely does not require the other.

## Read Lag

CQRS's most important limitation — the write succeeded, but the read model is behind. This is **[[Eventual Consistency|eventual consistency]]**, specifically a failure of immediate **read-your-writes** behavior (see also [[Replication Lag]]).

> [!warning] Failure modes & trade-offs
> **Eventual consistency UX**: Users submit a form, immediately reload the page, and don't see their own change. Common Staff-level mitigations:
> 1. **Return the new state or version token** from the command response so the UI updates optimistically or polls until `projection_version >= write_version`.
> 2. **Route read-your-writes** for the mutating user briefly to the write store (or pass a session watermark).
> 3. **Never validate new writes against the read projection** — always enforce business invariants against the authoritative write store.

## Materialized Projection

- **Projection:** a representation derived from other data.
- **Materialized:** calculated and stored ahead of time, rather than recalculated for every request.
- **CQRS:** using a read model that can differ from the write model.

> A materialized CQRS projection is a stored, query-friendly read model maintained from authoritative business data or events.

A usual implementation: listen for domain events (or [[Event Sourcing]] streams / [[Change Data Capture]]), update an aggregated table on each change. Useful for solving problems of [[BFF]] and [[Aggregator]] (solves request amplification, downstream availability coupling, slowest-response-time-wins, cross-service dependencies, etc.).

**The main cost: stale data.** The projection is not authoritative for accepting a new business operation.

### How do we update it safely?

A projection consumer needs the patterns we already covered:
- **[[Idempotent Consumer|Idempotency]]:** duplicate events must not apply an effect twice.
- **Ordering ([[Version Guard]]):** old events must not overwrite newer state.
- **Local transactions:** projection changes and processing progress commit together.
- **[[Reconciliation]]:** detect and repair missed or incorrect updates.

A subtle point: an overview combining several services usually does not have one universal sequence number.

You may track separate source progress:

```
last_order_sequence
last_payment_sequence
last_shipping_sequence
```

Sequence 12 from Order Service is not necessarily related to sequence 12 from Payment Service.

### Is it a database materialized view?

Not necessarily.

A [[PostgreSQL]] **materialized view** is a specific database feature storing the result of a query. It is typically refreshed explicitly.

A CQRS materialized projection is a broader architectural concept. It can be:
- An ordinary PostgreSQL table.
- An Elasticsearch index.
- A [[Redis]] structure.
- A database materialized view, where suitable.

Most event-driven, cross-service projections are application-maintained tables or documents.

CQRS (Command Query Responsibility Segregation):::Splitting a system's write model (commands, normalized/transactional) from its read model (queries, denormalized/query-optimized), synced asynchronously via events or CDC.

## Does CQRS require Event Sourcing? #card
**No.** CQRS only separates read and write models. The write side can remain a standard state-based relational DB (e.g., [[PostgreSQL]]) synced to read stores via [[Change Data Capture]] (CDC) or the [[Outbox Pattern]]. [[Event Sourcing]] is an orthogonal choice about storing write-side state as an append-only event log.

## How do you handle CQRS read lag (missing read-your-writes) in the UI? #card
1. Return the committed state or **version token** from the command so the client updates optimistically or waits until `projection_version >= write_version`.
2. Temporarily route the mutating user's immediate read to the authoritative write store.
3. Never use the stale read projection to validate new write commands.

In a multi-service CQRS projection, track ==per-source sequence numbers== (e.g., `last_order_sequence`, `last_payment_sequence`) because independent services have no global ordering. #card-reminder

## Related
- [[Event Sourcing]]
- [[Eventual Consistency]]
- [[Change Data Capture]]
- [[Outbox Pattern]]
- [[BFF]]
- [[Aggregator]]
- [[Version Guard]]
- [[Idempotent Consumer]]
- [[Reconciliation]]
- [[OLTP and OLAP]]
