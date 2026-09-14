**Core topic:** Data serialization and compatibility  
**Central question:** How can independently changing components continue exchanging and interpreting data correctly?

This chapter is highly related to the  **Evolvability** topic of the [[Reliability, Scalability, Maintainability#Maintainability]]

## C04.1 — Encoding formats

- In-memory objects versus serialized representations.
- Language-specific serialization.
- JSON, XML, and CSV.
- Binary encodings.
- Representation size, parsing cost, interoperability, and type fidelity.
- Security risks of unsafe deserialization.

A living application is constantly changing, and data schemas change. But for server-side application you cannot change data schema/format atomically - many instances might be running, and a rolling update might keep the system in incompatible state for some time. On the client side upgrade you have no guarantee when the user will update the up, so in worst case you have to support backward compatibility indefinitely:
- [[Backward Compatibility]]
- [[Forward Compatibility]]

Usually data lives in two forms: in-memory and serialized: [[Data Encoding]]



---

**C04.2 — Schema-based binary formats**
- Apache Thrift.
- Protocol Buffers.
- Apache Avro.
- Field names versus field tags.
- Writer schemas and reader schemas.
- Schema resolution and defaults.
- Code generation and dynamically generated schemas.

**C04.3 — Compatibility and evolution**
- Backward compatibility: newer code reading older data.
- Forward compatibility: older code reading newer data.
- Adding, removing, and changing fields.
- Optional fields and default values.
- Rolling upgrades and mixed-version deployments.
- Long-lived stored data outlasting the code that created it.

**C04.4 — Modes of dataflow**
- Dataflow through databases.
- Dataflow through services.
- REST and remote procedure calls: RPC.
- Remote calls versus local function calls.
- Message brokers and asynchronous communication.
- Distributed actor systems.

### Worth mentioning
- Compatibility depends on both the format and the particular schema change.
- A database can act as a communication channel between different application versions.
- RPC cannot remove network uncertainty simply by looking like a local function call.
- Serialization choices affect operational flexibility, not just payload size.
- Messaging decouples participants but still requires compatible message contracts.

**IT mapping:** API contracts; schema registries; service integration; event schemas; rolling deployment; backward-compatible migrations; serialization security.

**Connections:** `C02` schema assumptions; `C05` replication dataflow; `C08` remote-call failures; `C11` event-stream compatibility.