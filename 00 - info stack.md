> [!info]
> Info stack is the prioritized Senior-to-Staff learning backlog and inbox. Completed topics move into their domain folders and link back here.
>
> Extracted notes: [[Ansible]], [[Terraform]], [[Ansible vs Terraform]].

## Tier 1 — Distributed Execution & Workflow Primitives (Immediate)

Core building blocks behind durable orchestrators ([[Saga]], [[Temporal]], [[Outbox Pattern]], [[Reconciliation]]):

- [ ] [[Saga State Table]] — state-machine schema (`status`, `step`, `version`, `payload`, `updated_at`), optimistic transitions with [[Version Guard]], and auditability
- [ ] [[Worker Leases]] & [[Fencing Token|Fencing Tokens]] — lease acquisition (`lease_owner`, `lease_until`), heartbeat renewal, zombie-worker fencing on GC pauses / network partitions
- [ ] Claim counters & poison-pill isolation — bounded `attempt_count`, max-delivery cutoffs, and routing exhausted jobs to [[DLQ]]
- [ ] Retry timestamps & backoff scheduling — `next_run_at` indexing, exponential backoff with jitter ([[Retry Pattern]]), and avoiding thundering herds
- [ ] Scheduled polling vs event-driven wakeup — `SELECT ... FOR UPDATE SKIP LOCKED` batch claimers vs [[Change Data Capture]] / listen-notify push
- [ ] [[Timeout Pattern|Timeout handling]] & crash recovery — stuck-in-progress sweeper loops, idempotent re-execution ([[Idempotency Key]], [[Idempotent Consumer]]), and compensating actions

## Tier 2 — DDIA & Distributed Systems Core

Deepening [[Designing Data Intensive Applications]] and [[Learning path]] Phase 5 gaps:

- [ ] [[Linearizability]] — recency guarantees, CAP trade-offs ([[CAP Theorem]]), and why linearizable registers are expensive
- [ ] [[Snapshot Isolation]] & [[Write Skew]] — [[MVCC]] anomalies, phantom reads, and Serializable Snapshot Isolation (SSI) vs 2PL
- [ ] Distributed consensus: [[Raft]], [[Paxos]], [[Quorum Consensus]], and [[Split-Brain]] prevention
- [ ] [[Two-Phase Commit]] (2PC) internals — coordinator crash blocking, XA transactions, and exact contrast with [[2PC vs Saga]]
- [ ] [[Consistent Hashing]] & dynamic rebalancing — virtual nodes, hot-key mitigation, and [[Partitioning and Secondary Indexes]]
- [ ] [[Change Data Capture]] (CDC) & stream processing — Debezium WAL log tailing, replication slot disk exhaustion, and [[Event-driven architecture]]
- [ ] [[Distributed Lock|Distributed Locks]] — Redlock vs consensus-backed locks (`[[etcd]]` / ZooKeeper) and why TTL locks alone are unsafe without [[Fencing Token|fencing tokens]]

## Tier 3 — JVM & Backend Stack Mastery

Hands-on depth for production Java/PostgreSQL/distributed data systems:

- [ ] **Java 21** — [[Virtual Threads]] (pinning pitfalls with `synchronized`, carrier thread pools), structured concurrency, scoped values, ZGC/G1GC tuning ([[Java versions]], [[jvmArgs]])
- [ ] **Java simple cache implementation** — bounded LRU/W-TinyLFU eviction, lock striping vs `ConcurrentHashMap`, cache stampede / single-flight coalescing, TTL vs memory-bounded sizing ([[concurrency]])
- [ ] **[[PostgreSQL]] (via jOOQ)** — type-safe SQL generation, CTEs/[[Window Functions]], query plan analysis (`EXPLAIN (ANALYZE, BUFFERS)`), `autovacuum` & HOT updates, connection pool sizing via [[Little's Law]]
- [ ] **Flyway** — expand-and-contract zero-downtime [[Schema Migration]], `lock_timeout`, `CREATE INDEX CONCURRENTLY`, checksum drift, and rollback strategies
- [ ] **[[Redis]]** — single-threaded event loop stalls, persistence (`RDB` vs `AOF` + fork CoW memory spikes), Lua atomic scripts, cluster hash slots, and eviction policies
- [ ] **Amazon DynamoDB** — single-table design, partition/sort keys, GSI/LSI trade-offs, adaptive capacity, conditional writes, and DynamoDB Streams
- [ ] **Riak** — [[Leaderless Replication]], dotted version vectors (causal context), [[Sloppy Quorum]], [[CRDT]] data types, and [[Anti-Entropy]]
- [ ] **[[Event-driven architecture]]** — schema registry & [[Backward Compatibility|backward]]/[[Forward Compatibility|forward compatibility]], [[Kafka]] EOS vs consumer idempotency, out-of-order event handling

## Tier 4 — Architecture, Reliability & Platform

Cross-cutting Staff-level production engineering:

- [ ] [[Rate Limiting]] & [[Backpressure]] — token bucket / sliding window algorithms, distributed limiters, load shedding, and end-to-end flow control
- [ ] [[OpenTelemetry]] & observability — structured logging, RED/USE metrics, tail-sampled [[Distributed Tracing]], and SLO error budgets
- [ ] [[Service Mesh]] & [[Zero-Trust Security]] — mTLS identity rotation, L7 traffic policies, and avoiding double-resilience traps with application-level [[Circuit Breaker]] / [[Retry Pattern]]
- [ ] **[[Kubernetes]] internals** — CNI networking, `kube-proxy`/eBPF `EndpointSlice` drain races, HPA/VPA/KEDA autoscaling, and Pod Disruption Budgets
- [ ] **GCP core architecture** — IAM hierarchy, VPC networking, GKE, Cloud SQL / Spanner, Pub/Sub, and Cloud Run

## Raw Scratchpad / Unsorted

- [ ] How DB schema migration libraries work under the hood (see [[Learning path]])
- [ ] Passing data between services using DB anti-patterns vs [[Outbox Pattern]] / [[Change Data Capture]]
