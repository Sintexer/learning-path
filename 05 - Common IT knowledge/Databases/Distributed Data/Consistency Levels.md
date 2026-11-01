
Distributed systems store data on many computers (nodes). A consistency model is a contract. It tells how and when all nodes show the same data. 

### 1. Strict Consistency (Linearizability)

* **Definition:** This is the strongest consistency model.
* **How it works:** 
  * The system acts like one single machine.
  * When a client writes data, all other clients see the new data immediately.
  * The system uses real-world time to order all operations.
* **Result:** No client can ever read old data.

Because networks drop packets and computers have out-of-sync clocks, achieving this is one of the hardest problems in computer science.

In modern distributed systems, linearizability is achieved through a combination of **four core architectural pillars**
#### **Consensus Algorithms (A Single Total Order)**.

To ensure that all nodes agree on the exact order of events, systems use consensus protocols like Raft or Multi-Paxos:
- **The Replicated State Machine (RSM):**   Linearizable databases do not replicate the _data_ directly; they replicate an append-only log of _actions_. If every node starts at State 0 and executes the exact same log of operations in the exact same order, they are guaranteed to reach the exact same state.
- **Quorums:** A write does not need to wait for every node in the cluster. It only needs an acknowledgment from a majority (a quorum):   $Quorum=⌊N2⌋+1$  . Because any two majorities must overlap by at least one node, the system guarantees that the most up-to-date data is always present in any future quorum.
#### Solving the Read Path (Avoiding "Split-Brain" Stale Reads)

Consensus on writes is not enough. If a cluster experiences a network partition, an old leader might not realize it has been deposed. If it answers read requests, it will serve stale data, breaking linearizability.

To fix this, linearizable systems implement strict read protocols:
- **Quorum Reads (Read-Index):**  
    Before the leader answers a read request, it sends a heartbeat to a quorum of nodes to confirm: _"Am I still the leader right now?"_ Once the quorum confirms it, the leader serves the read.
- **Leader Leases:**  
    To avoid network round-trips for every read, nodes grant the leader a time-bound "lease" (e.g., for 500 milliseconds). During this window, followers guarantee they will not elect a new leader. The leader can serve reads locally with the absolute certainty that it is the only leader.

#### Synchronized Time & "Commit Wait" (Scaling Across Regions)

A single Raft or Paxos group cannot scale globally because a single leader in New York cannot serve low-latency writes to Tokyo. To distribute data globally while preserving strict real-world time ordering, systems use specialized clock mechanisms:
- **TrueTime (Google Spanner):**  
    Spanner uses atomic clocks and GPS receivers installed in every datacenter. Because physical clocks drift, TrueTime represents time not as a single number, but as a bounded range: $[now−ϵ,now+ϵ]$, where $ϵ$ (uncertainty) is usually under a few milliseconds.
- **The "Commit Wait" Pattern:**  
    When a node writes data at timestamp T, the database intentionally **delays the write acknowledgment** to the client. It waits for 2ϵ time to pass.  
    By pausing execution, the database guarantees that the actual, physical wall-clock time everywhere in the world has passed timestamp T. Thus, any subsequent read anywhere on Earth will mathematically receive a timestamp greater than T.

#### Cross-Shard Coordination (Two-Phase Commit + Consensus)

Large systems partition (shard) data across many consensus groups. To perform transactions across multiple shards linearly:
1. **Two-Phase Commit (2PC):** A coordinator manages the transaction across shards (Prepare phase → Commit phase).
2. **Consensus underneath 2PC:** Historically, 2PC was vulnerable if the coordinator died. Modern linearizable databases (like CockroachDB, YugabyteDB, and Spanner) make every participant and the coordinator itself a **Raft or Paxos group**. If a coordinator node dies, the consensus group elects a new one, preventing the system from freezing.

### 2. Sequential Consistency

* **Definition:** Operations obey a sequence, but not real-world time.
* **How it works:**
  * All nodes must see all operations in the same order.
  * The operations of each program run in the order that the program specifies.
  * The system can delay operations, but it cannot change the order of operations.
* **Result:** All nodes see the same history of events.

### 3. Causal Consistency

* **Definition:** Cause and effect determine the order of operations.
* **How it works:**
  * If event B happens because of event A, then event A is a cause. 
  * All nodes must see event A before they see event B.
  * For example, a question is a cause, and the reply is the effect.
  * If two events do not have a relation, nodes can show them in different orders.
* **Result:** Connected events stay in the correct order.

### 4. Client-Centric Consistency Levels

These models give guarantees to one client session, but not to the whole system.

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

### 5. Eventual Consistency

* **Definition:** This is the weakest consistency model.
* **How it works:**
  * If no person makes new updates, all nodes will finally become the same.
  * The system does not guarantee when this will occur.
  * Replicas can return different data at the same time.
* **Result:** The system is very fast and available, but clients can read old data.