---
flashcards:
  q-nxkj: { nid: 1795356717841, hash: iu7ff3z7, sync: d8ker9tj }
---

#learn

CAP theorem states that a distributed system can't provide more than two of these three guarantees simultaneously:

- **Consistency ([[Linearizability]])**: All clients see the same, most recent data at the same time no matter which node they connect to (every read receives the most recent write or an error).
- **[[Availability]]**: Every request received by a non-failing node gets a non-error response (without guaranteeing it contains the most recent write).
- **Partition Tolerance**: A partition indicates a communication break between nodes ([[Split-Brain]] risk). Partition tolerance means the system continues to operate despite arbitrary message loss or network partitions.

## Why "Pick 2 of 3" Is Misleading

In any real networked distributed system, **network partitions (P) happen** — you cannot opt out of P. So the real choice *during a network partition* is **CP** (reject writes/reads on the minority side to preserve [[Linearizability]], e.g., [[etcd]] / [[Raft]] / [[Paxos]]) vs. **AP** (keep serving potentially stale or divergent data and reconcile later via [[Eventual Consistency]], [[CRDT]], [[Read Repair]], or [[Anti-Entropy]], e.g., [[Apache Cassandra]] with [[Sloppy Quorum]]).

1. Picking a database is not easy. Justifying our choice purely based on the CAP theorem is not enough. For example, companies don't choose [[Apache Cassandra]] for chat applications simply because it is an AP system. There is a list of characteristics (write throughput via [[LSM Tree]], tunable [[Consistency Levels]], partition-key access patterns) that make Cassandra desirable. We need to dig deeper.
2. "CAP prohibits only a tiny part of the design space: perfect availability and consistency in the presence of partitions, which are rare." — *CAP Twelve Years Later: How the "Rules" Have Changed* (Eric Brewer).
3. The theorem is about 100% availability and strict [[Linearizability]]. A more realistic day-to-day discussion is the **PACELC theorem**:
   - **P**artition → choose **A**vailability vs. **C**onsistency.
   - **E**lse (normal operation, no partition!) → choose **L**atency vs. **C**onsistency (e.g., [[Synchronous vs Asynchronous Replication]]).

> [!warning] Failure modes & trade-offs
> **CAP "C" ≠ ACID "C"**: In CAP, *Consistency* specifically means **[[Linearizability]]** (single-object recency across replicas), NOT [[ACID]]'s business-invariant consistency (foreign keys, balance >= 0). Also, many real databases don't sit rigidly in one bucket — e.g., [[Apache Cassandra]] lets you tune [[Quorum Consensus]] (`W + R > N`) per query.

**Is the CAP theorem actually useful?**

It is useful as a starting point for tradeoff discussions, especially around [[Split-Brain]] and [[Replication]], but PACELC and concrete [[Consistency Levels]] are what matter in production design.

## What does the PACELC theorem add to the CAP theorem? #card
CAP only describes the trade-off **during a network partition** (Availability vs. Consistency). **PACELC** adds what matters 99.9% of the time when there is **no partition (Else)**: the trade-off between **Latency** and **Consistency** (e.g., synchronous quorum replication vs. fast asynchronous local reads).

What specific form of consistency does the "C" in CAP refer to?::[[Linearizability]] (every read returns the most recent completed write or an error across all nodes) — not [[ACID]] invariant consistency.

In a real distributed system you cannot choose "CA" because ==network partitions== are unavoidable; the real choice during a partition is ==CP vs. AP==.
^q-nxkj

## Related
- [[Linearizability]]
- [[Availability]]
- [[Consistency Levels]]
- [[Eventual Consistency]]
- [[Split-Brain]]
- [[Quorum Consensus]]
- [[Raft]]
- [[Paxos]]
- [[Replication]]
- [[Synchronous vs Asynchronous Replication]]
- [[Sloppy Quorum]]
- [[Apache Cassandra]]
- [[etcd]]
