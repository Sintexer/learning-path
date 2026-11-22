---
flashcards:
  q-msdx: { nid: 1795361722031, hash: smue5jhn, sync: mgpujxu8 }
  q-fg4z: { nid: 1795361722054, hash: s6pnjek7, sync: ze8taikx }
---

**Linearizability** (also called *atomic consistency* or *strong consistency*) is the strongest single-object consistency model: the distributed system behaves as if there were **only one copy of the data**, and every operation takes effect atomically at some point between its invocation and response.

Once a write completes, **all subsequent reads by any client must see that value (or a newer one)**, respecting real-world time ordering.

## Four Architectural Pillars of Linearizability

### 1. Consensus Algorithms (A Single Total Order)

Nodes agree on a single total order of operations via [[Raft]] or [[Paxos|Multi-Paxos]] (used in [[etcd]], ZooKeeper, CockroachDB):
- **Replicated State Machine (RSM)**: Nodes replicate an append-only log of deterministic *actions*. Starting from the same initial state and applying the same log in the same order guarantees identical state.
- **[[Quorum Consensus|Quorums]]**: A write succeeds once acknowledged by a majority ($\lfloor N/2 \rfloor + 1$). Any two majorities overlap by at least one node, guaranteeing the latest committed state is visible to future quorums.

### 2. Linearizable Read Path (Preventing [[Split-Brain]] Stale Reads)

Consensus on writes alone is insufficient: during a network partition, a deposed leader might not know a new leader was elected and could serve stale local reads. Systems prevent this with:
- **Quorum Reads (Read-Index)**: Before answering a read, the leader confirms via heartbeat to a quorum that it is still leader and has applied up to the current commit index.
- **Leader Leases**: Followers promise not to elect a new leader during a bounded lease window (e.g., 500 ms), letting the leader serve reads locally without an extra round-trip. Safety depends on bounded clock drift and GC pauses (pair with [[Fencing Token|fencing tokens]] for external side effects).

### 3. Synchronized Time & Commit Wait (Global Scale)

A single [[Raft]] group cannot give low-latency global writes. Google Spanner solves global real-time ordering with:
- **TrueTime**: Atomic clocks + GPS in every datacenter return a bounded time interval $[now - \epsilon, now + \epsilon]$ instead of a single timestamp.
- **Commit Wait**: After assigning write timestamp $T$, the database delays acknowledging the write for $2\epsilon$, guaranteeing real-world time everywhere has passed $T$ before any subsequent read begins.

### 4. Cross-Shard Coordination ([[Two-Phase Commit]] + Consensus)

When data is [[Data partitioning|partitioned]] across multiple consensus groups, linearizable multi-shard transactions run **[[Two-Phase Commit]]** where every shard *and* the coordinator is itself a [[Raft]]/[[Paxos]] group — preventing coordinator-crash blocking.

> [!warning] Failure modes & trade-offs
> - **CAP & PACELC cost**: Under a network partition, a linearizable system must sacrifice availability ([[CAP Theorem]]). Even *without* partitions, linearizability pays a latency penalty proportional to network round-trips (PACELC).
> - **Linearizability ≠ Serializability**: Linearizability is a *recency* guarantee on a **single object**; Serializability is an *isolation* guarantee across **multi-object [[Transaction|transactions]]**. Combining both produces **Strict Serializability** (external consistency).
> - **Leader lease clock-drift hazard**: A long JVM stop-the-world GC pause or NTP clock jump on a lease-holding leader can cause it to serve stale reads after its lease expired.

## What are the two main mechanisms linearizable consensus systems use to prevent a deposed leader from serving stale reads? #card
1. **Quorum Read (Read-Index)**: The leader sends a heartbeat round to a majority quorum before replying to confirm it is still the active leader.
2. **Leader Lease**: Followers promise not to start an election for a bounded lease duration, allowing fast local reads while assuming bounded clock drift and pause times.

Under the PACELC theorem, linearizability pays a cost even when there is no network partition: it trades ==higher read/write latency== for single-object recency guarantees.
^q-msdx

Linearizability is required for leader election, uniqueness constraints, and distributed locks; avoid paying its latency cost for feeds, analytics, or counters where causal or eventual consistency suffices. #card-reminder
^q-fg4z

## Related
- [[Consistency Levels]]
- [[CAP Theorem]]
- [[Raft]]
- [[Paxos]]
- [[Quorum Consensus]]
- [[Two-Phase Commit]]
- [[Split-Brain]]
- [[Fencing Token]]
- [[etcd]]
- [[Eventual Consistency]]
