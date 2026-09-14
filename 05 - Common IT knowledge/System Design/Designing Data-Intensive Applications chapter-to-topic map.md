# Part I — Foundations of Data Systems

**Group theme:** Understanding application requirements, data representation, storage engines, and communication between components.

## C01 — Reliable, Scalable, and Maintainable Applications

**Core topic:** Data-system quality attributes  
**Central question:** What makes a data-intensive application successful beyond simply producing correct results?

### Grouped subtopics

**C01.1 — Applications as combinations of data components**
- Databases, caches, search indexes, message queues, and stream/batch processors.
- Combining components into a larger application-level data system.
- Responsibility for correctness when data crosses component boundaries.
- Choosing tools according to workloads and requirements.

**C01.2 — Reliability and fault tolerance**
- Faults versus user-visible system failures.
- Hardware faults: disks, memory, power, and machine failures.
- Software faults: systematic bugs, cascading failures, and resource exhaustion.
- Human errors and operational mistakes.
- Redundancy, isolation, testing, monitoring, and recovery.
- Deliberately introducing faults to test resilience.

**C01.3 — Scalability and workload characterization**
- Load parameters: request rates, read/write ratios, data volume, and workload skew.
- Throughput and response-time measurements.
- Latency versus response time.
- Percentiles and tail latency rather than averages alone.
- Scaling up versus scaling out.
- Elastic systems versus manually provisioned capacity.
- Workload-dependent architecture, illustrated through social-feed fan-out.

**C01.4 — Maintainability**
- Operability: making systems manageable in production.
- Simplicity: reducing accidental complexity.
- Evolvability: accommodating new requirements.
- Abstraction as a tool for controlling complexity.

### Worth mentioning
- Scalability is not an absolute property; it describes how a system handles a particular kind of growth.
- Averages can conceal serious user-facing performance problems.
- Fault tolerance means coping with faults, not assuming faults can be eliminated.
- Reliability requirements depend on the consequences and costs of failure.

**IT mapping:** Nonfunctional requirements; SRE; production readiness; capacity planning; performance testing; observability; chaos engineering; technical-debt management.

**Connections:** `C03` storage performance; `C05–C06` scaling through distribution; `C08` failure behavior; `C12` system-wide correctness.

---

## C02 — Data Models and Query Languages

**Core topic:** Logical data modeling and data access  
**Central question:** How do data models influence application code, relationships, and the queries a system can express?

### Grouped subtopics

**C02.1 — Relational and document models**
- Relational tables, rows, keys, and joins.
- Documents, nested structures, and one-to-many relationships.
- Object-relational impedance mismatch.
- Data locality and retrieving related data together.
- Normalization and denormalization.
- Many-to-one and many-to-many relationships.
- Application complexity when joins move out of the database.

**C02.2 — Schema and data evolution**
- Schema-on-write versus schema-on-read.
- Explicitly enforced schemas versus implicitly assumed structures.
- Heterogeneous records and changing application requirements.
- Migration and compatibility implications of model choices.

**C02.3 — Query-language styles**
- Declarative queries versus imperative instructions.
- SQL and optimizer-controlled execution.
- Declarative expression of selection, joining, and aggregation.
- MapReduce-style querying.
- How query abstractions affect optimization and parallel execution.

**C02.4 — Graph data models**
- Property graphs: vertices, edges, and properties.
- Graph traversal and relationship-heavy queries.
- Cypher.
- Triple stores and RDF.
- SPARQL.
- Datalog and rule-based querying.

**C02.5 — Historical model trade-offs**
- Hierarchical and network databases.
- Navigational access versus relational querying.
- Why earlier data-model limitations remain relevant to newer systems.

### Worth mentioning
- “Schemaless” generally means the schema is not centrally enforced, not that no schema exists.
- Document databases are not automatically superior for all object-oriented applications.
- Graph, document, and relational models optimize different ways of representing relationships.
- Declarative languages allow execution strategies to change without rewriting the query’s intent.

**IT mapping:** Domain modeling; database selection; SQL versus document databases; ORM design; schema governance; graph databases; knowledge graphs; query optimization.

**Connections:** `C03` physical representation; `C04` schema compatibility; `C10–C11` alternative query-execution models.

---

## C03 — Storage and Retrieval

**Core topic:** Storage-engine architecture and access paths  
**Central question:** How do databases physically store data, and why are different engines suitable for different workloads?

### Grouped subtopics

**C03.1 — Log-structured storage**
- Append-only data files.
- Hash indexes.
- Segments, compaction, and merging.
- Sorted String Tables: SSTables.
- Memtables and Log-Structured Merge trees: LSM trees.
- Bloom filters.
- Background compaction and its performance consequences.

**C03.2 — Page-oriented storage**
- B-trees and B-tree pages.
- Tree traversal, page updates, and page splitting.
- Write-ahead logging for crash recovery.
- Concurrency and implementation considerations.
- B-trees versus LSM trees.
- Read, write, and space amplification trade-offs.

**C03.3 — Index structures and specialized access**
- Primary and secondary indexes.
- Clustered and nonclustered storage.
- Heap files and covering indexes.
- Multi-column indexes.
- Multidimensional and spatial indexes.
- Full-text search and fuzzy matching.
- In-memory databases and their durability mechanisms.

**C03.4 — Transactional versus analytical workloads**
- Online Transaction Processing: OLTP.
- Online Analytical Processing: OLAP.
- Data warehouses.
- Extract–Transform–Load: ETL.
- Fact and dimension tables.
- Star and snowflake schemas.

**C03.5 — Column-oriented analytical storage**
- Column-oriented layouts.
- Compression and bitmap encoding.
- Sort order within column stores.
- Vectorized processing and memory efficiency.
- Handling writes in column-oriented systems.
- Materialized aggregates and data cubes.

### Worth mentioning
- An index improves some reads but introduces write, storage, and maintenance costs.
- A log-structured storage engine is not the same concept as an application event log.
- Keeping data in memory does not necessarily mean abandoning durability.
- Column-oriented storage and “wide-column” database models are different concepts.
- Materialized views trade computation at read time for storage and maintenance work.

**IT mapping:** Database internals; index tuning; query performance; write amplification; search-engine internals; data warehousing; analytical databases; storage-engine selection.

**Connections:** `C02` logical versus physical models; `C05` replication logs; `C07` transaction implementation; `C10–C11` building derived datasets.

---

## C04 — Encoding and Evolution

**Core topic:** Data serialization and compatibility  
**Central question:** How can independently changing components continue exchanging and interpreting data correctly?

### Grouped subtopics

**C04.1 — Encoding formats**
- In-memory objects versus serialized representations.
- Language-specific serialization.
- JSON, XML, and CSV.
- Binary encodings.
- Representation size, parsing cost, interoperability, and type fidelity.
- Security risks of unsafe deserialization.

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

---

# Part II — Distributed Data

**Group theme:** Distributing data across machines while managing scaling, concurrency, failures, and agreement.

## C05 — Replication

**Core topic:** Maintaining multiple copies of data  
**Central question:** How can replicas improve availability and performance without creating unacceptable consistency problems?

### Grouped subtopics

**C05.1 — Leader-based replication**
- Leaders and followers.
- Synchronous and asynchronous replication.
- Setting up new followers.
- Catch-up recovery.
- Leader failure and failover.
- Statement-based, write-ahead-log, logical, and trigger-based replication.

**C05.2 — Replication lag and read guarantees**
- Eventual consistency.
- Read-your-writes consistency.
- Monotonic reads.
- Consistent-prefix reads.
- User-visible anomalies caused by stale or differently delayed replicas.
- Application-level strategies for obtaining stronger reads.

**C05.3 — Multi-leader replication**
- Multi-datacenter writes.
- Offline clients.
- Collaborative editing.
- Write conflicts.
- Conflict avoidance and resolution.
- Last-write-wins and preserving concurrent versions.
- Replication topologies and ordering complications.

**C05.4 — Leaderless replication**
- Reads and writes sent to multiple replicas.
- Read and write quorums.
- Sloppy quorums and hinted handoff.
- Read repair and anti-entropy.
- Detecting stale values.
- Limitations of quorum-based consistency.

**C05.5 — Concurrent writes and causality**
- Happens-before relationships.
- Concurrent versus causally ordered writes.
- Version tracking and version vectors.
- Merging concurrent values.
- Why physical timestamps alone do not reliably establish causality.

### Worth mentioning
- Replication topology and consistency guarantees are separate design choices.
- A quorum-overlap formula is not, by itself, a guarantee of linearizability.
- Asynchronous failover can lose acknowledged writes.
- Last-write-wins can discard legitimate concurrent updates.
- Concurrent events do not need to occur at exactly the same physical time.

**IT mapping:** High availability; read replicas; disaster-recovery architecture; multi-region databases; offline synchronization; conflict resolution; distributed consistency.

**Connections:** `C06` partitioned replication; `C08` failover uncertainty; `C09` stronger coordination; `C11` change streams.

---

## C06 — Partitioning

**Core topic:** Dividing datasets and workloads across nodes  
**Central question:** How can a system distribute data efficiently while avoiding hotspots and excessive coordination?

### Grouped subtopics

**C06.1 — Partitioning strategies**
- Partitioning, also called sharding.
- Key-range partitioning.
- Hash partitioning.
- Combining hashing with ordered or composite keys.
- Relationships between partitioning and replication.
- Partition skew and hotspots.

**C06.2 — Workload skew**
- Hot keys and disproportionately popular records.
- Why even hash distribution does not guarantee even workload distribution.
- Splitting a hot key’s workload through application-level techniques.
- Additional complexity introduced into reads and aggregation.

**C06.3 — Secondary indexes over partitions**
- Document-partitioned, or local, secondary indexes.
- Scatter/gather queries.
- Term-partitioned, or global, secondary indexes.
- Read efficiency versus write coordination.
- Asynchronous index maintenance.

**C06.4 — Rebalancing**
- Moving partitions when capacity changes.
- Fixed numbers of partitions.
- Dynamic partition splitting.
- Partition allocation proportional to node count.
- Why simple hash-modulo-node-count schemes cause excessive movement.
- Automatic versus operationally controlled rebalancing.

**C06.5 — Request routing**
- Finding the node responsible for a key.
- Routing through any node.
- Dedicated routing tiers.
- Partition-aware clients.
- Coordination services for partition metadata.
- Parallel query execution across partitions.

### Worth mentioning
- Partitioning divides responsibility; replication duplicates responsibility.
- A balanced dataset can still have an unbalanced request workload.
- The partition key influences query efficiency, transaction boundaries, and future scalability.
- Secondary indexes make partitioning substantially more complicated than basic key-value routing.
- Hash partitioning and consistent hashing are related but not interchangeable terms.

**IT mapping:** Database sharding; partition-key design; hotspot mitigation; distributed indexes; cluster rebalancing; routing metadata; distributed query planning.

**Connections:** `C03` indexing; `C05` replicas within partitions; `C07` cross-partition transactions; `C09` coordination; `C10` shuffles.

---

## C07 — Transactions

**Core topic:** Correctness under concurrent access and failures  
**Central question:** Which guarantees do transactions provide, and which anomalies remain at different isolation levels?

### Grouped subtopics

**C07.1 — Transaction guarantees**
- Atomicity, Consistency, Isolation, Durability: ACID.
- Single-object and multi-object operations.
- Aborting and retrying transactions.
- Application invariants.
- Database guarantees versus application responsibilities.

**C07.2 — Weak isolation and observed anomalies**
- Dirty reads.
- Dirty writes.
- Read committed isolation.
- Nonrepeatable reads and read skew.
- Snapshot isolation.
- Multi-Version Concurrency Control: MVCC.

**C07.3 — Concurrent update problems**
- Lost updates.
- Atomic update operations.
- Explicit locking.
- Compare-and-set.
- Automatic detection of conflicting updates.
- Interactions between concurrency control and replication.

**C07.4 — Write skew and phantoms**
- Invariants involving multiple records.
- Concurrent decisions based on the same apparent state.
- Predicate-based reads.
- Phantom records.
- Turning abstract conflicts into explicit lockable records.

**C07.5 — Serializability implementations**
- Actual serial execution.
- Stored procedures and partition-local execution.
- Two-phase locking: 2PL.
- Shared and exclusive locks.
- Predicate and index-range locking.
- Serializable Snapshot Isolation: SSI.
- Optimistic conflict detection and transaction aborts.

### Worth mentioning
- ACID “consistency” concerns application invariants; it is not the same as replica consistency.
- Atomicity does not, by itself, provide isolation.
- Snapshot isolation prevents many anomalies but does not generally prevent write skew.
- Serializable execution need not physically execute transactions one at a time.
- Retrying a transaction is dangerous when it includes nontransactional external side effects.
- Isolation-level names can mean different things in different databases.

**IT mapping:** Transaction design; race conditions; database isolation configuration; optimistic concurrency control; locking; invariant enforcement; financial and inventory correctness.

**Connections:** `C03` storage mechanisms; `C05` replicated writes; `C09` distributed commit and ordering; `C12` end-to-end correctness.

---

## C08 — The Trouble with Distributed Systems

**Core topic:** Failure models, uncertainty, and time  
**Central question:** What assumptions become unsafe when computation spans independently failing machines?

### Grouped subtopics

**C08.1 — Partial failure**
- Some components failing while others continue.
- Nondeterministic failure observations.
- A request succeeding even when its response is lost.
- Cloud and shared-network environments.
- Differences between distributed applications and tightly controlled computing environments.

**C08.2 — Unreliable networks**
- Packet loss, delay, and reordering.
- Connection failures.
- Queueing and network congestion.
- Timeouts and failure detection.
- Slow nodes being indistinguishable from failed nodes.
- Trade-offs in timeout selection.

**C08.3 — Unreliable clocks**
- Time-of-day clocks versus monotonic clocks.
- Clock drift and synchronization.
- NTP adjustments and clock discontinuities.
- Timestamp ordering problems.
- Clock uncertainty.

**C08.4 — Process pauses and ownership**
- Garbage-collection pauses.
- Scheduling delays and suspended processes.
- Leases expiring while a process is paused.
- Stale processes continuing to act.
- Fencing tokens that prevent outdated owners from writing.

**C08.5 — Distributed-system models**
- Synchronous, partially synchronous, and asynchronous timing assumptions.
- Crash-stop and crash-recovery failures.
- Byzantine faults.
- Quorums and how a system establishes authority.
- Safety properties versus liveness properties.

### Worth mentioning
- A timeout indicates uncertainty, not proof that the remote operation failed.
- A lease or distributed lock is insufficient if downstream systems accept writes from expired owners.
- Wall-clock timestamps should not automatically be treated as a reliable global order.
- Safety means something bad does not happen; liveness means useful progress eventually occurs.
- Correctness claims depend on an explicitly stated failure model.

**IT mapping:** Fault injection; timeout design; failure detectors; distributed locking; fencing; clock synchronization; resilience engineering; formal system assumptions.

**Connections:** `C05` failover; `C07` retries; `C09` consensus assumptions; `C11` event time and fault recovery.

---

## C09 — Consistency and Consensus

**Core topic:** Agreement, ordering, and strong consistency  
**Central question:** How can distributed components agree on values and decisions despite failures?

### Grouped subtopics

**C09.1 — Linearizability**
- Making an operation appear to occur at one instant.
- Respecting real-time ordering.
- Linearizable registers and compare-and-set.
- Uses in coordination, locking, and uniqueness.
- Linearizability versus serializability.
- Latency and availability costs of strong consistency.

**C09.2 — CAP and network partitions**
- Network partitions.
- Linearizable behavior versus serving every request during a partition.
- Why CAP is not a general-purpose “pick any two” product classification.
- Consistency trade-offs outside the simple CAP framing.

**C09.3 — Causality and ordering**
- Causal order versus total order.
- Sequence numbers.
- Lamport timestamps.
- Why assigning timestamps is different from knowing that all earlier events have arrived.
- Total-order broadcast.
- Relationships between ordering and consensus.

**C09.4 — Distributed commit**
- Atomic commit across participants.
- Two-phase commit: 2PC.
- Prepare and commit phases.
- Coordinator failure and in-doubt transactions.
- Blocking and recovery.
- XA and heterogeneous distributed transactions.
- Operational limitations of distributed commit.

**C09.5 — Consensus algorithms and coordination services**
- Agreement, validity, integrity, and termination.
- Leader election and leader epochs.
- Quorum-based decisions.
- Paxos, Raft, Viewstamped Replication, and Zab.
- Limits imposed by the FLP result.
- Membership, configuration, and coordination services.
- ZooKeeper-style coordination use cases.

### Worth mentioning
- Serializability orders transactions; linearizability additionally constrains operations relative to real time. Neither term should substitute for the other.
- Causal consistency does not require a total order over unrelated events.
- Two-phase commit and consensus address related but different problems.
- FLP does not mean useful consensus systems cannot be built; practical systems rely on additional assumptions for progress.
- Coordination services are generally used for small amounts of critical metadata, not all application data.

**IT mapping:** Strongly consistent databases; distributed coordination; leader election; configuration management; atomic commit; uniqueness enforcement; consensus-backed metadata stores.

**Connections:** `C05` replication guarantees; `C06` membership and routing; `C07` transaction semantics; `C08` failure assumptions; `C12` coordination alternatives.

---

# Part III — Derived Data

**Group theme:** Producing indexes, views, analytics, and other useful representations from existing data while preserving correctness.

## C10 — Batch Processing

**Core topic:** Bounded data processing and recomputation  
**Central question:** How can large datasets be transformed reliably and efficiently into new datasets?

### Grouped subtopics

**C10.1 — Batch dataflow and the Unix model**
- Transforming input datasets into output datasets.
- Composable processing stages.
- Standard interfaces and separation of concerns.
- Sorting, grouping, filtering, and joining.
- Immutable inputs and reproducible processing.

**C10.2 — MapReduce and distributed filesystems**
- Map and reduce tasks.
- Distributed filesystems such as HDFS.
- Data locality.
- Partitioning and shuffle.
- Sorting reducer input.
- Task scheduling and failed-task recovery.

**C10.3 — Distributed joins and aggregation**
- Reduce-side joins.
- Sort-merge joins.
- Broadcast hash joins.
- Partitioned hash joins.
- Grouping and aggregation.
- Handling skewed keys and uneven work distribution.

**C10.4 — Outputs and ecosystem design**
- Building search indexes.
- Producing key-value datasets.
- Materializing derived data.
- Hadoop-style storage and schema-on-read.
- Comparisons with massively parallel analytical databases.
- Separating storage formats from processing logic.

**C10.5 — Beyond classic MapReduce**
- Dataflow DAGs.
- Reducing unnecessary intermediate materialization.
- Pipelining and in-memory execution.
- Spark, Tez, and Flink.
- Iterative graph processing.
- Pregel and bulk synchronous parallel execution.
- Higher-level languages and query optimization.

### Worth mentioning
- Batch processing is not synonymous with MapReduce.
- Materialized intermediate results have a cost, but can simplify fault recovery.
- Recomputing derived data can be safer than maintaining complicated mutable state.
- Data partitioning and ordering often determine join performance.
- External side effects complicate the otherwise clean retry semantics of batch tasks.

**IT mapping:** ETL/ELT pipelines; data lakes; distributed analytics; backfills; data transformations; distributed joins; search-index construction; reproducible data processing.

**Connections:** `C02` query languages; `C03` warehouses and indexes; `C06` partitioning; `C11` unbounded processing; `C12` rebuildable data systems.

---

## C11 — Stream Processing

**Core topic:** Continuous processing of event data  
**Central question:** How can systems derive useful state from ongoing event streams while handling ordering, time, and failures?

### Grouped subtopics

**C11.1 — Messaging and event transport**
- Events, producers, and consumers.
- Direct messaging versus brokers.
- Queues and publish/subscribe.
- Acknowledgments, redelivery, and consumer failures.
- Load balancing versus fan-out.
- Message ordering and processing concurrency.

**C11.2 — Log-based messaging**
- Durable append-only logs.
- Partitioned logs.
- Consumer offsets.
- Replay and retention.
- Contrasts with brokers that remove messages after acknowledgment.
- Ordering guarantees within partitions.

**C11.3 — Databases as event sources**
- Change Data Capture: CDC.
- Database snapshots followed by change streams.
- Log compaction.
- Event sourcing.
- Immutable events and derived current state.
- Command Query Responsibility Segregation: CQRS.
- Maintaining caches, indexes, and materialized views.

**C11.4 — Continuous computation and time**
- Complex event processing.
- Stream analytics and aggregation.
- Event time versus processing time.
- Out-of-order and late events.
- Window completion and watermark-like progress information.
- Tumbling, hopping, sliding, and session windows.

**C11.5 — Stream joins and fault tolerance**
- Stream–stream joins.
- Stream–table joins.
- Table–table joins.
- Stateful operators.
- Microbatching and checkpointing.
- Recovery through replay.
- Idempotent processing.
- Atomic updates and exactly-once effects.

### Worth mentioning
- A durable log and a work queue serve different consumption and retention patterns.
- Ordering within one partition does not imply global ordering.
- CDC exposes data changes; event sourcing records application events as the primary model.
- Exactly-once claims require a defined boundary: broker delivery, operator state, and external side effects are different concerns.
- Time-dependent joins can become nondeterministic if replay does not reproduce the relevant historical state.
- Immutable history introduces retention and deletion challenges.

**IT mapping:** Event-driven architecture; streaming platforms; CDC pipelines; event sourcing; CQRS; real-time analytics; incremental view maintenance; stateful stream processing.

**Connections:** `C04` event contracts; `C05` logs and replication; `C08` clocks and retries; `C10` batch equivalence; `C12` integrated dataflow architectures.

---

## C12 — The Future of Data Systems

**Core topic:** Composing correct, evolvable, and responsible data systems  
**Central question:** How can multiple specialized components behave as one coherent application-level data system?

### Grouped subtopics

**C12.1 — Data integration and derived data**
- Systems of record and derived representations.
- Keeping databases, caches, indexes, and analytical stores synchronized.
- Problems caused by independent dual writes.
- Using ordered change streams to propagate updates.
- Rebuilding derived datasets from source data.
- Combining specialized storage and processing tools.

**C12.2 — Unbundling databases**
- Separating storage, indexing, query processing, and maintenance.
- Comparing composed data systems with integrated databases.
- Dataflow as a way to connect components.
- Application code participating in view maintenance.
- Trade-offs between log-based propagation and distributed transactions.

**C12.3 — Unifying batch and stream processing**
- Historical recomputation alongside continuous updates.
- Lambda architecture and its operational complexity.
- Shared logic for bounded and unbounded processing.
- Reprocessing historical events.
- Evolving transformations without losing correctness.

**C12.4 — End-to-end correctness**
- Correctness across the full request-to-result path.
- Duplicate suppression and unique request identifiers.
- Exactly-once effects beyond individual components.
- Enforcing constraints in asynchronous systems.
- Coordination requirements for operations such as uniqueness checks.
- Workflows, compensation, and handling business-rule violations.
- Distinguishing timeliness from integrity.

**C12.5 — Trust, verification, and auditability**
- Verifying derived data.
- Reconciliation and integrity checking.
- Reproducible processing.
- Auditable dataflows.
- Detecting corruption and implementation mistakes.
- Designing systems whose results can be independently checked.

**C12.6 — Ethics and societal consequences**
- Predictive analytics and automated decision-making.
- Bias and discrimination.
- Privacy, surveillance, and informed consent.
- Data ownership and power asymmetries.
- Retention and control over personal data.
- Responsibilities of people building data systems.

### Worth mentioning
- This chapter is a synthesis and architectural argument, not merely a catalog of newer tools.
- System-wide correctness cannot be inferred from the guarantees of each component in isolation.
- Asynchronous propagation can delay visibility without necessarily corrupting the underlying information.
- An immutable log is an architectural choice, not permission to retain personal data indefinitely.
- Technical correctness and socially acceptable behavior are separate requirements.

**IT mapping:** Data-platform architecture; event-driven integration; materialized-view maintenance; rebuildable read models; reconciliation; auditability; privacy engineering; responsible data use.

**Connections:** Synthesizes the whole book, especially `C04`, `C07`, and `C09–C11`.