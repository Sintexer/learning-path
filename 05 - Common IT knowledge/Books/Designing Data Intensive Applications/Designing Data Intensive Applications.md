Master index for Martin Kleppmann's *Designing Data-Intensive Applications* (DDIA). For the detailed chapter-by-chapter syllabus, see [[Designing Data-Intensive Applications chapter-to-topic map]].

## Part I — Foundations of Data Systems
1. **Chapter 1** — [[Reliability, Scalability, Maintainability]]
2. **Chapter 2 — Data Models and Query Languages**
   - [[Relational Database]]
   - [[Document Database]]
   - [[SQL vs NoSQL]]
   - [[Impedance mismatch]]
   - [[Query Languages for Data]]
   - [[MapReduce querying]]
   - [[Graph database]]
3. **Chapter 3** — [[DDIA Chapter 3 Storage and Retrieval]]
   - [[Database Storage Engines]], [[Database Log]], [[WAL]], [[LSM Tree]], [[SSTable]], [[B-Tree Index]], [[Column-Oriented Storage]], [[OLTP and OLAP]]
4. **Chapter 4** — [[DDIA Chapter 4 Encoding and Evolution]]
   - [[Data Encoding]], [[Backward Compatibility]], [[Forward Compatibility]], [[Protocol Buffers]], [[Apache Thrift]]

## Part II — Distributed Data
5. **Chapter 5 — [[Replication]]**
   - [[Leader-Based Replication]], [[Synchronous vs Asynchronous Replication]], [[Replication Log]], [[Replication Lag]]
   - [[Multi-Leader Replication]], [[Leaderless Replication]], [[Sloppy Quorum]], [[Read Repair]], [[Anti-Entropy]], [[CRDT]], [[Types of Writes in distributed systems]]
6. **Chapter 6 — [[Data partitioning|Partitioning]]**
   - [[Replication vs Partitioning]], [[Partitioning and Secondary Indexes]], [[Consistent Hashing]]
7. **Chapter 7 — [[Transaction|Transactions]]**
   - [[ACID]], [[MVCC]], [[Snapshot Isolation]], [[Write Skew]], [[Optimistic Locking]], [[Pessimistic Locking]]
8. **Chapter 8 — The Trouble with Distributed Systems**
   - [[Fencing Token]], [[Split-Brain]], [[Distributed Lock]], [[CAP Theorem]]
9. **Chapter 9 — Consistency and Consensus**
   - [[Consistency Levels]], [[Eventual Consistency]], [[Linearizability]], [[Raft]], [[Paxos]], [[Two-Phase Commit]], [[Quorum Consensus]]

## Part III — Derived Data
10. **Chapter 10 — Batch Processing** — [[MapReduce querying]], [[ETL]]
11. **Chapter 11 — Stream Processing** — [[Kafka]], [[Change Data Capture]], [[Event Sourcing]], [[CQRS]], [[Delivery Semantics]]
12. **Chapter 12 — The Future of Data Systems** — [[Dual-Write]], [[Outbox Pattern]], [[Reconciliation]], [[Idempotency]]

## Related
- [[Designing Data-Intensive Applications chapter-to-topic map]]
- [[Learning path]]
- [[Distributed Data]]
