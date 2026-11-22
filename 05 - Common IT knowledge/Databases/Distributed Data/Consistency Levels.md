---
flashcards:
  q-fdt5: { nid: 1795356725842, hash: 4btfh2nw, sync: sy3d78un }
  q-2s4v: { nid: 1795356725868, hash: zv5bth5i, sync: ex62bywq }
---

Distributed systems store data on many computers (nodes). A **consistency model** is a contract that defines how and when all nodes show the same data (see also [[CAP Theorem]]).

### 1. Strict Consistency ([[Linearizability]])

* **Definition:** This is the strongest single-object consistency model.
* **How it works:**
  * The system acts like one single copy of the data.
  * Once a write completes, all subsequent reads by any client must see that value (or a newer one).
  * The system respects real-world time ordering of operations.
* **Result:** No client can ever read stale data after a write completes.

Because networks drop packets and computers have out-of-sync clocks, achieving [[Linearizability|linearizability]] is one of the hardest problems in computer science.

In modern distributed systems, linearizability is achieved through a combination of **four core architectural pillars**:

#### Consensus Algorithms (A Single Total Order)

To ensure that all nodes agree on the exact order of events, systems use consensus protocols like [[Raft]] or [[Paxos|Multi-Paxos]] (used in [[etcd]], ZooKeeper, CockroachDB):
- **The Replicated State Machine (RSM):** Linearizable databases do not replicate the *data* directly; they replicate an append-only log of *actions*. If every node starts at State 0 and executes the exact same log of operations in the exact same order, they are guaranteed to reach the exact same state.
- **[[Quorum Consensus|Quorums]]:** A write does not need to wait for every node in the cluster. It only needs an acknowledgment from a majority (a quorum): $\text{Quorum} = \lfloor N/2 \rfloor + 1$. Because any two majorities must overlap by at least one node, the system guarantees that the most up-to-date data is always present in any future quorum.

#### Solving the Read Path (Avoiding "[[Split-Brain]]" Stale Reads)

Consensus on writes is not enough. If a cluster experiences a network partition, an old leader might not realize it has been deposed. If it answers read requests locally, it will serve stale data, breaking [[Linearizability|linearizability]].

To fix this, linearizable systems implement strict read protocols:
- **Quorum Reads (Read-Index):** Before the leader answers a read request, it sends a heartbeat to a quorum of nodes to confirm: *"Am I still the leader right now?"* Once the quorum confirms it, the leader serves the read.
- **Leader Leases:** To avoid network round-trips for every read, nodes grant the leader a time-bound lease (e.g., 500 ms). During this window, followers guarantee they will not elect a new leader, so the leader can serve reads locally. (Relies on bounded clock drift — if the leader's clock runs slow or experiences a long GC pause, leases can break safety unless paired with [[Fencing Token|fencing tokens]].)

#### Synchronized Time & "Commit Wait" (Scaling Across Regions)

A single [[Raft]] or [[Paxos]] group cannot scale globally because a single leader in New York cannot serve low-latency writes to Tokyo. To distribute data globally while preserving strict real-world time ordering, systems use specialized clock mechanisms:
- **TrueTime (Google Spanner):** Spanner uses atomic clocks and GPS receivers installed in every datacenter. Because physical clocks drift, TrueTime represents time not as a single number, but as a bounded range: $[now - \epsilon, now + \epsilon]$, where $\epsilon$ (uncertainty) is usually under a few milliseconds.
- **The "Commit Wait" Pattern:** When a node writes data at timestamp $T$, the database intentionally **delays the write acknowledgment** to the client for $2\epsilon$. By pausing, the database guarantees that actual physical wall-clock time everywhere on Earth has passed timestamp $T$, so any subsequent read anywhere receives a timestamp greater than $T$.

#### Cross-Shard Coordination ([[Two-Phase Commit]] + Consensus)

Large systems [[Data partitioning|partition (shard)]] data across many consensus groups. To perform transactions across multiple shards linearly:
1. **[[Two-Phase Commit]] (2PC):** A coordinator manages the transaction across shards (Prepare phase → Commit phase).
2. **Consensus underneath 2PC:** Historically, 2PC was vulnerable if the coordinator died. Modern linearizable databases (like CockroachDB, YugabyteDB, and Spanner) make every participant and the coordinator itself a **[[Raft]] or [[Paxos]] group**. If a coordinator node dies, the consensus group elects a new one, preventing the system from blocking.

### 2. Sequential Consistency

* **Definition:** Operations obey a single global sequence, but not necessarily real-world time.
* **How it works:**
  * All nodes must see all operations in the same order.
  * The operations of each individual client/program run in the order that program specifies.
  * The system can delay operations relative to real time, but it cannot reorder them differently on different nodes.
* **Result:** All nodes see the same history of events.

### 3. Causal Consistency

* **Definition:** Cause and effect determine the order of operations (see [[Types of Writes in distributed systems]]).
* **How it works:**
  * If event B happens because of event A (happens-before), then event A is a cause.
  * All nodes must see event A before they see event B.
  * For example, a question is a cause, and the reply is the effect.
  * If two events are concurrent (unrelated), nodes can show them in different orders.
* **Result:** Connected events stay in the correct order without paying the full latency cost of [[Linearizability|linearizability]].

### 4. Client-Centric Consistency Levels

These models give guarantees to one client session (see [[Replication Lag]]), but not globally across all clients.

#### A. Read-Your-Writes Consistency
* When you write data, you will immediately see that data on your next read.
* Other users can see the old data for a short time.

#### B. Monotonic Reads
* You will never see older data than the data you saw before.
* Time does not move backward during your reads.

#### C. Monotonic Writes
* The system processes your write operations in the order that you send them.
* The system does not mix the order of your writes.

#### D. Writes-Follow-Reads
* You read a value, and then you write a new value.
* Other users will see the first value before they see your new value.

---

### 5. [[Eventual Consistency]]

* **Definition:** This is the weakest consistency model.
* **How it works:**
  * If no new updates are made, all replicas will eventually converge to the same value.
  * The system does not guarantee *when* this will occur.
  * Replicas can return different data at the same time.
* **Result:** The system is very fast and available, but clients can read stale data.

> [!warning] Failure modes & trade-offs
> **Linearizability ≠ Serializability**:
> - **[[Linearizability]]** is a *recency* guarantee on a **single object** (real-time ordering).
> - **Serializability** is an *isolation* property of **multi-object [[Transaction|transactions]]** (equivalent to some serial order, not necessarily real-time).
> - Combining both gives **Strict Serializability** (external consistency, as in Spanner). Do not conflate them in design reviews.

## Why is consensus on writes alone not enough to guarantee linearizable reads? #card
During a network partition, a deposed leader might not know a new leader was elected. If it serves reads from its local state, it returns stale data ([[Split-Brain]]). Linearizable reads require either a **quorum heartbeat (Read-Index)** before replying or a time-bounded **Leader Lease**.
^q-fdt5

## What is the difference between Linearizability and Serializability? #card
- **[[Linearizability]]**: Recency guarantee on reads/writes of a **single object** respecting real-world time.
- **Serializability**: [[Transaction]] isolation guarantee across **multiple objects** (transactions execute as if in *some* serial order, not necessarily real-time).
Combining both yields **Strict Serializability**.

In Google Spanner, the **Commit Wait** pattern delays acknowledging a write for ==2ε (twice the clock uncertainty bound)== so that real-world time everywhere has passed the write's timestamp before any subsequent read can start.
^q-2s4v

## Related
- [[Linearizability]]
- [[Eventual Consistency]]
- [[CAP Theorem]]
- [[Raft]]
- [[Paxos]]
- [[Quorum Consensus]]
- [[Two-Phase Commit]]
- [[Split-Brain]]
- [[Fencing Token]]
- [[Replication Lag]]
- [[Types of Writes in distributed systems]]
