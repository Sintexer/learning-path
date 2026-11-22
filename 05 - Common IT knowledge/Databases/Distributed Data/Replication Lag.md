---
aliases:
  - Asynchronous Replication Lag
---
**Replication lag** is the delay between a write happening on the leader and being reflected on a follower in [[Synchronous vs Asynchronous Replication|asynchronous]] [[Leader-Based Replication]]. In normal operation it may be a fraction of a second and unnoticeable, but under heavy load or network issues it can grow to seconds or minutes.

It is the primary driver of [[Eventual Consistency]] in read-scaling architectures.

## Problems with Replication Lag (and How to Mitigate Them)

### 1. Read-Your-Own-Writes ([[Consistency Levels#A. Read-Your-Writes Consistency|Read-Your-Writes Consistency]])
With asynchronous replication, if a user views data shortly after submitting a write, the read may hit a lagging follower and make it look like their update was lost.
- **Mitigations**:
  - Read things the user *can modify* (e.g., their own profile) from the leader; read others' profiles from followers.
  - Track the timestamp of the user's last update and route reads to the leader for ~1 minute afterward, or pass the logical write timestamp (LSN) and ensure the chosen follower is caught up to at least that LSN.
  - **Cross-device wrinkle**: If the user writes on desktop and reads on mobile, metadata must be centralized and both devices must route to the same datacenter.

### 2. Monotonic Reads ([[Consistency Levels#B. Monotonic Reads|Monotonic Reads]])
A user refreshing a page hits a fast follower (sees a new comment), then on the next refresh hits a lagging follower (the comment vanishes — **time appears to move backward**).
- **Mitigation**: Pin each user to the **same replica** (e.g., hash of `user_id` selects the follower) so reads never jump backward in time, falling back only if that replica fails.

### 3. Consistent Prefix Reads
If a sequence of writes happens in a causal order (e.g., a question followed by an answer), a reader observing partitioned/sharded replicas with different lag might see the **answer before the question**.
- **Mitigation**: Ensure causally related writes are written to the **same partition** (or explicitly track causality via version vectors).

> [!warning] Failure modes & trade-offs
> When application code starts accumulating complex workarounds for replication lag (sticky routing, LSN tracking, selective leader reads), you are effectively re-implementing weak [[Transaction|transactions]] at the application layer. If your domain cannot tolerate stale reads, consider routing critical reads to the leader or adopting a strongly consistent store.

## How do you prevent "time moving backward" (violating Monotonic Reads) when reading from async followers? #card
Route all reads for a given user consistently to the **same replica** (e.g., by hashing the user ID to pick a follower), so they never jump from a caught-up follower to a lagging follower.

## Name two practical techniques to guarantee Read-Your-Writes consistency with async read replicas. #card
1. Route reads of user-editable data (or all reads for ~1 minute after a user's write) to the **leader**.
2. Track the write's replication log position (LSN/timestamp) on the client and only serve reads from followers that have caught up to at least that position.

**Consistent prefix reads** prevent a reader from seeing ==an effect before its cause (e.g., an answer before the question)== when causally related writes live on differently lagging partitions.

## Related
- [[Leader-Based Replication]]
- [[Synchronous vs Asynchronous Replication]]
- [[Eventual Consistency]]
- [[Consistency Levels]]
- [[Replication]]
- [[Replication Log]]
