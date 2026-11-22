---
flashcards:
  q-vvga: { nid: 1795356717891, hash: asu95en7, sync: g6hdcwxw }
  q-uffe: { nid: 1795356717917, hash: cii5kn9t, sync: 9ftts32d }
---

**Event-driven architecture (EDA)** is a design paradigm where services communicate by producing and consuming events. This approach decouples services and allows them to react to changes asynchronously.

An event is a change in state, a user request, or an update, like an item being placed in a shopping cart in an e-commerce website. When an event occurs, the information is published for other services to consume it.

In event-driven architectures, events are the primary mechanism for sharing information across services. These events are **observable facts** ("OrderPlaced") rather than directed commands ("ChargeCard").

## Key Concepts

- **Events**: Represent immutable facts about significant changes or actions within a system (e.g., `OrderPlaced`, `UserRegistered`).
- **Event Producers**: Services that generate and publish events (reliably via [[Outbox Pattern]] or [[Change Data Capture]] to avoid [[Dual-Write]]).
- **Event Consumers**: Services that listen for and react to events (safely via [[Idempotent Consumer]] and [[Version Guard]]).
- **Event Brokers**: Middleware that durably stores and routes events from producers to consumers (e.g., [[Kafka]], RabbitMQ).

## Common Event Payload Styles

1. **Event Notification (thin event)**: Carries minimal metadata (`orderId`, `eventType`). Consumers call back to the producer's API if they need full details. Simple schema evolution, but creates read-back load coupling.
2. **Event-Carried State Transfer (ECST)**: Carries enough state in the payload so consumers can maintain local read copies ([[CQRS]]) without calling back. Reduces runtime coupling, but increases payload size and schema governance ([[Backward Compatibility]] / [[Forward Compatibility]]).
3. **[[Event Sourcing]]**: Events themselves are the write-side source of truth, not just integration messages.

## Benefits

- **Loose Coupling**: Producers don't need to know who consumes their events or when.
- **Scalability & Load Leveling**: Brokers buffer traffic spikes and let consumers scale independently (see [[Kafka Partition]] and [[Backpressure]]).
- **Resilience**: Services can continue accepting writes even if downstream consumers are temporarily down.

> [!warning] Failure modes & trade-offs
> - **Hidden coupling & "pinball architecture"**: Pure choreography makes end-to-end business flows hard to trace, debug, or evolve without [[Distributed Tracing]] and clear domain ownership (see [[Microservices Orchestration and Choreography]]).
> - **Delivery & ordering hazards**: Brokers provide [[At-least-once delivery]], not magic exactly-once. Consumers must handle duplicates ([[Idempotent Consumer]]), out-of-order replays ([[Version Guard]]), and poison pills ([[DLQ]]).
> - **Schema evolution**: Changing an event payload breaks unknown downstream consumers unless you enforce [[Backward Compatibility]] (e.g., [[Protocol Buffers]] / Avro + a Schema Registry).

## Event Notification (thin event) vs. Event-Carried State Transfer (ECST) — what is the trade-off? #card
- **Thin Event Notification** (`orderId` only): Small payloads and simpler schema evolution, but forces consumers to call back to the producer API (runtime availability & load coupling).
- **Event-Carried State Transfer (ECST)**: Consumers stay autonomous and can build local [[CQRS]] views without callbacks, at the cost of larger payloads, eventual staleness, and stricter schema compatibility.
^q-vvga

Event-driven producers must publish via ==Outbox Pattern or Change Data Capture== to avoid [[Dual-Write]] loss, and consumers must handle ==duplicates, reordering, and poison pills==. #card-reminder
^q-uffe

## Related
- [[Kafka]]
- [[Microservices Orchestration and Choreography]]
- [[Event Sourcing]]
- [[CQRS]]
- [[Outbox Pattern]]
- [[Dual-Write]]
- [[Delivery Semantics]]
- [[Idempotent Consumer]]
- [[DLQ]]
- [[Backpressure]]
- [[DDD]]
