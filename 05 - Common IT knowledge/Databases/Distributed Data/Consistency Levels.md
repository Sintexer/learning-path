---
flashcards:
  q-fdt5: { nid: 1795356725842, hash: 4btfh2nw, sync: sy3d78un }
  q-2s4v: { nid: 1795356725868, hash: zv5bth5i, sync: fbu7sn5y }
---

Distributed systems store data on many computers (nodes). A **consistency model** is a contract that defines how and when all nodes show the same data (see also [[CAP Theorem]]).

### 1. Strict Consistency ([[Linearizability]])

* **Definition:** The strongest single-object consistency model — the system behaves as if there were one single copy of the data, respecting real-world time ordering.
* **Result:** Once a write completes, all subsequent reads by any client see that value (or a newer one); no stale reads are possible.
* **How it is built:** Consensus ([[Raft]] / [[Paxos]]), linearizable reads (Quorum Read-Index or Leader Leases), synchronized time (Spanner TrueTime + Commit Wait), and consensus-backed [[Two-Phase Commit]]. See **[[Linearizability]]** for the full architectural deep dive.

### 2. Sequential Consistency

* **Definition:** Operations obey a single global sequence, but not necessarily real-world time.
* **How it works:**
  * All nodes see all operations in the same order.
  * Operations of each individual client/program execute in program order.
  * The system may delay operations relative to wall-clock time, but cannot reorder them differently on different nodes.
* **Result:** All nodes observe the same global history.

### 3. Causal Consistency

* **Definition:** Cause and effect determine the order of operations (see [[Types of Writes in distributed systems]]).
* **How it works:**
  * If event B happens because of event A (happens-before), all nodes see event A before event B (e.g., a question before its reply).
  * Concurrent (unrelated) events may appear in different orders on different nodes.
* **Result:** Causally linked events stay ordered without paying the full latency cost of [[Linearizability|linearizability]].

### 4. Client-Centric Consistency Levels

These models give guarantees to one client session (see [[Replication Lag]]), not globally across all clients.

#### A. Read-Your-Writes Consistency
* When you write data, you immediately see that data on your next read (other users may still see old data briefly).

#### B. Monotonic Reads
* You never see older data than data you already read — time does not move backward during your session.

#### C. Monotonic Writes
* The system processes your writes in the exact order you send them.

#### D. Writes-Follow-Reads
* After you read a value and write a new value, other users see the first value before your new value.

---

### 5. [[Eventual Consistency]]

* **Definition:** The weakest consistency model.
* **How it works:** If no new updates arrive, all replicas eventually converge to the same value, with no bound on *when*.
* **Result:** Fast and highly available, but clients can observe stale or diverging values in the meantime.

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
