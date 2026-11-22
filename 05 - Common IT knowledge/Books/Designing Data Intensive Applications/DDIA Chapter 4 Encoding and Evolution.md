**Core topic:** Data serialization and compatibility
**Central question:** How can independently changing components continue exchanging and interpreting data correctly?

This chapter is highly related to the **Evolvability** topic of [[Reliability, Scalability, Maintainability#Maintainability]].

## C04.1 — Encoding formats

- In-memory objects versus serialized representations.
- Language-specific serialization.
- JSON, XML, and CSV.
- Binary encodings.
- Representation size, parsing cost, interoperability, and type fidelity.
- Security risks of unsafe deserialization.

A living application is constantly changing, and data schemas change. But for server-side applications you cannot change a data schema/format atomically — many instances might be running, and a [[Zero-downtime deployment#Rolling update|rolling update]] keeps the system in a mixed-version state for some time. On the client side you have no guarantee when the user will update the app, so in the worst case you have to support [[Backward Compatibility|backward compatibility]] indefinitely:
- [[Backward Compatibility]]
- [[Forward Compatibility]]

Usually data lives in two forms: in-memory and serialized: [[Data Encoding]].

---

## C04.2 — Schema-based binary formats
- [[Apache Thrift]]
- [[Protocol Buffers]]
- [[Apache Avro]]
- Field names versus field tags.
- Writer schemas and reader schemas (Avro).
- Schema resolution and defaults.
- Code generation and dynamically generated schemas.

## C04.3 — Compatibility and evolution
- [[Backward Compatibility]]: newer code reading older data.
- [[Forward Compatibility]]: older code reading newer data.
- Adding, removing, and changing fields.
- Optional fields and default values.
- Rolling upgrades and mixed-version deployments.
- Long-lived stored data outlasting the code that created it (see [[Schema Migration]]).

## C04.4 — Modes of dataflow
- **Dataflow through databases**: writing process encodes, reading process decodes later (often a newer or older version during rolling deploys).
- **Dataflow through services**: [[REST]] and RPC ([[gRPC]]); servers are usually updated before clients, so requests need backward compatibility and responses need forward compatibility.
- **Remote calls versus local function calls**: network calls are unpredictable (timeouts, lost responses, retries requiring [[Idempotency]]).
- **Message brokers and asynchronous communication**: [[Kafka]] / [[Event-driven architecture]] decouple sender and receiver; publishers and consumers evolve independently.
- **Distributed actor systems**: message-passing concurrency across nodes.

### Worth mentioning
- Compatibility depends on both the format and the particular schema change.
- A database can act as a communication channel between different application versions.
- RPC cannot remove network uncertainty simply by looking like a local function call.
- Serialization choices affect operational flexibility, not just payload size.
- Messaging decouples participants but still requires compatible message contracts.

> [!warning] Failure modes & trade-offs
> **Location transparency fallacy in RPC**: Frameworks that hide network calls behind local function syntax obscure timeouts, partial failures, retries, and latency differences. Always treat remote calls as fallible — enforce explicit [[Timeout Pattern|timeouts]], [[Retry Pattern|retries]] with [[Idempotency Key|idempotency keys]], and [[Circuit Breaker|circuit breakers]].

## Why is trying to make RPC look identical to a local function call fundamentally flawed? #card
Unlike local function calls, network requests can fail unpredictably, time out without telling you whether the remote operation succeeded, be duplicated on retry (requiring [[Idempotency]]), suffer wildly variable latency, and cross language/schema boundaries.

During a typical server-first deployment, API requests require ==backward compatibility== (new server reads old client requests) and API responses require ==forward compatibility== (old client reads new server responses).

**IT mapping:** [[API]] contracts; [[API versioning]]; schema registries; service integration; event schemas; [[Zero-downtime deployment|rolling deployment]]; [[Schema Migration|backward-compatible migrations]]; serialization security.

**Connections:** `C02` schema assumptions; `C05` [[Replication]] dataflow; `C08` remote-call failures; `C11` event-stream compatibility.

## Related
- [[Data Encoding]]
- [[Backward Compatibility]]
- [[Forward Compatibility]]
- [[Protocol Buffers]]
- [[Apache Thrift]]
- [[gRPC]]
- [[REST]]
- [[Schema Migration]]
- [[Designing Data-Intensive Applications chapter-to-topic map]]
