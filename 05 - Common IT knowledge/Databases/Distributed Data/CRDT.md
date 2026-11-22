**CRDTs (Conflict-free Replicated Data Types)** are specialized distributed data structures that can be updated independently and concurrently on multiple replicas without coordination, and are mathematically guaranteed to merge deterministically to the exact same state once all updates are delivered.

They are used in [[Multi-Leader Replication]], [[Leaderless Replication]], collaborative editors, and offline-first apps to achieve **Strong Eventual Consistency** without losing concurrent updates (unlike Last-Write-Wins).

## Mathematical Foundation

For replicas to converge regardless of network reordering, duplication, or delay, the merge operation $\sqcup$ must form a **join-semilattice** — meaning it is:
1. **Commutative**: $A \sqcup B = B \sqcup A$ (order of arrival does not matter).
2. **Associative**: $(A \sqcup B) \sqcup C = A \sqcup (B \sqcup C)$ (grouping does not matter).
3. **Idempotent**: $A \sqcup A = A$ (duplicate messages are harmless; aligns with [[At-least-once delivery]]).

## Two Flavors of CRDTs

- **State-based (CvRDT — Convergent)**: Replicas periodically send their *full state* (or delta state) to peers and merge via $\sqcup$. Requires only an unreliable, unordered, at-least-once network, at the cost of larger payloads.
- **Operation-based (CmRDT — Commutative)**: Replicas broadcast *operations* (deltas). Smaller network payloads, but requires reliable causal broadcast (no duplicates, causal delivery order).

## Common CRDT Types

- **G-Counter (Grow-only Counter)**: One slot per replica; each replica increments only its own slot; merge takes element-wise `max`; value is the sum.
- **PN-Counter (Positive-Negative Counter)**: Two G-Counters — one for increments (`P`), one for decrements (`N`); value is `sum(P) - sum(N)`.
- **OR-Set (Observed-Remove Set)**: Tags each added element with a unique ID; remove deletes only observed tags, so a concurrent `add` and `remove` of the same item has well-defined semantics (add-wins) without resurrecting old deletes.
- **LWW-Element-Set / RGA / Yjs / Automerge**: Used for ordered sequences and collaborative text editing.

> [!warning] Failure modes & trade-offs
> - **Metadata & tombstone growth**: Tracking per-replica counters or unique add/remove tags causes metadata to grow over time (and tombstones are hard to garbage-collect safely without knowing all replicas have seen them).
> - **Cannot enforce global invariants**: CRDTs cannot prevent business-rule violations that require coordination — e.g., preventing a bank balance from going negative or ensuring two users do not book the same seat. Those require [[Linearizability]], [[Quorum Consensus|consensus]] ([[Raft]]/[[Paxos]]), or [[Transaction|serializable transactions]].

## What three algebraic properties must a state-based CRDT merge function satisfy? #card
1. **Commutative**: `merge(A, B) = merge(B, A)` (handles out-of-order delivery).
2. **Associative**: `merge(merge(A, B), C) = merge(A, merge(B, C))` (handles arbitrary merge grouping).
3. **Idempotent**: `merge(A, A) = A` (handles duplicate delivery).

## When can you NOT use a CRDT for conflict resolution? #card
When the application must enforce a **global invariant** across concurrent writers — such as preventing double-booking a unique seat, enforcing unique usernames, or preventing an account balance from dropping below zero. Those require synchronous coordination (e.g., [[Linearizability]] or consensus).

A ==PN-Counter== CRDT supports both increments and decrements by combining two grow-only G-Counters (`P` and `N`) and computing `sum(P) - sum(N)`.

## Related
- [[Multi-Leader Replication]]
- [[Leaderless Replication]]
- [[Eventual Consistency]]
- [[Types of Writes in distributed systems]]
- [[Idempotency]]
- [[Linearizability]]
