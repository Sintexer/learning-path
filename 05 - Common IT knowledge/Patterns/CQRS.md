Command Query Responsibility Segregation. **CQRS = using separate models for writing data (commands) versus reading data (queries).**

That's it at its core — deceptively simple. But "separate models" can mean different things depending on how far you take it:

- At minimum: separate _code paths/classes_ for handling writes vs reads, even against the same database
- Taken further (much more common in real distributed systems): **separate data stores entirely** — a write-optimized store and a completely different, read-optimized store, kept in sync

### Why would you ever want this? The core motivation

Writes and reads have fundamentally different needs.

**The write side (commands) needs:**
- Strong consistency and validation ("can this bet actually be placed given the account balance and business rules?")
- Transactional integrity (money can't disappear or duplicate)
- A normalized structure that avoids data anomalies (classic relational design — foreign keys, constraints)

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

The write side handles a command, validates it, applies it to its own store. Then, **some mechanism** propagates that change into the read-optimized store, so queries stay up to date (usually with some small delay — this introduces **[[IT Common Glossary#Eventual consistency|eventual consistency]]** between write and read sides, which is a tradeoff you should be ready to name explicitly).

## CQRS does not require Event Sourcing

You can implement CQRS in a very simple way:

- Write side: a service writes directly to Postgres, normally, with regular `UPDATE`/`INSERT` statements — no event log, no special pattern, just a normal transactional write
- Sync mechanism: a background job periodically queries what changed and pushes it into the read store, **or** you use **Change Data Capture (CDC)** — a tool (like Debezium) that watches the Postgres write-ahead log directly and streams row-level changes out, without the application even being aware
- Read side: ClickHouse (or Elasticsearch, or a denormalized read replica, etc.) gets updated from that stream

**No [[Event Sourcing|event sourcing]] anywhere in this picture.** The write side never stores "the sequence of things that happened" — it just stores current state, like any normal database, and updates get propagated forward. This is a completely valid, very common, simpler form of CQRS.

[[Event Sourcing]] is a _different, additional_ decision about **how the write side itself stores its data** — and it happens to compose beautifully with CQRS, which is why they're so often mentioned in the same breath. But one absolutely does not require the other.

## Read Lag

CQRS’s Most Important Limitation - The write succeeded, but the read model is behind. This is **[[IT Common Glossary#Eventual consistency|eventual consistency]]**, specifically a failure of immediate **read-your-writes** behavior.

### The sentence to have ready if asked "does CQRS require event sourcing?"

> "No — CQRS is just about separating write and read models, which can be done with a plain database and a simple sync job or CDC pipeline. Event Sourcing is a separate decision about _how_ you store write-side state — as a log of events rather than current state. They compose very well together, which is why they're often discussed as a pair, but you can absolutely have one without the other."