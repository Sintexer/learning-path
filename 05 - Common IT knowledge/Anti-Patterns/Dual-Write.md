Writing to two independent systems where you need "both or neither," but have no atomic mechanism spanning both.

You have a service that needs to do **two things** when handling a request:

1. Update its own database (e.g., mark an order as `SHIPPED` in [[PostgreSQL]])
2. Publish an event to [[Kafka]] so other services know it happened (e.g., `OrderShipped` event, so a notifications service can email the customer)

These are **two independent systems** (Postgres and Kafka). You cannot wrap a database transaction and a Kafka produce call into one atomic operation — there's no distributed transaction spanning both by default. So you're left with an unavoidable ordering choice, and **both naive orderings are broken**:

**Option 1: Update DB first, then publish to Kafka.**  
What if the process crashes (or the network fails) _after_ the DB commit succeeds but _before_ the Kafka publish happens? The order is marked shipped in your database, but **no event was ever sent** — downstream services never find out. Silent inconsistency.

**Option 2: Publish to Kafka first, then update DB.**  
What if the Kafka publish succeeds, but the subsequent DB update fails or the process crashes right after? Now other services think the order shipped (they got the event), but **your own database says otherwise**. Even worse — an inconsistency that's now been broadcast to the rest of the system.

## Solution

Solution: Don't actually do two writes — do one write, and defer the second

The trick: instead of writing to your database **and separately** to Kafka, you write to your database **and an "outbox" table, in the same database, in the same transaction** - [[Outbox Pattern]].

Since both statements are in **one transaction, against one database**, this is a completely normal, fully atomic operation — either both rows change, or neither does. No distributed transaction needed at all, because you never actually touched two different systems in that step. The "event" is just another row in the same database.
