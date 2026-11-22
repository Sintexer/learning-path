In **Synchronous** [[Replication]], the leader must **block and wait** until 1..N followers confirm they have received and persisted the write before acknowledging success to the client.

Usually in [[Relational Database|relational databases]] (e.g., [[PostgreSQL]] `synchronous_commit`, MySQL semi-sync), only **one** follower is synchronous while the remaining followers are asynchronous — this is called **semi-synchronous** replication. It guarantees that at least two nodes (the leader + 1 follower) hold an up-to-date copy of every committed write, so if the leader dies, that synchronous follower can be promoted without data loss. Making *all* followers synchronous is impractical: any single follower outage or network hiccup would stall all writes across the entire cluster (violating [[Availability]]).

Often, [[Leader-Based Replication]] is configured to be completely **asynchronous** (the leader responds to the client as soon as it commits locally, without waiting for any follower). Asynchronous replication keeps write latency low and allows the leader to keep accepting writes even if all followers fall behind or go offline. However, if the leader fails and is unrecoverable, any writes not yet replicated to followers are permanently lost — weakening durability even for writes already acknowledged to the client.

> [!warning] Failure modes & trade-offs
> - **Tail latency in sync/semi-sync**: Write latency is bound by the slowest synchronous follower's network round-trip + disk `fsync`. Never place a synchronous follower across a high-latency WAN link unless strict zero-RPO cross-region durability justifies the latency hit.
> - **Semi-sync degradation**: If the single synchronous follower fails, the system must either promote an async follower to become the new synchronous follower immediately, or decide whether to block writes (prioritizing durability) vs temporarily fall back to async (prioritizing availability).
> - **Async failover data loss**: Under async replication, a client receiving `200 OK` does *not* guarantee survival across a leader crash (RPO > 0) and introduces [[Replication Lag]] anomalies on follower reads.

## Why is "semi-synchronous" replication preferred over making all followers synchronous? #card
Making all followers synchronous means a single slow or failed follower stalls all writes for the entire database. **Semi-synchronous** replication keeps 1 follower synchronous (guaranteeing an up-to-date copy survives leader failure) while keeping other followers asynchronous so the cluster stays available if a node fails.

## What is the durability risk of completely asynchronous leader-based replication? #card
The leader acknowledges writes to the client before followers receive them. If the leader crashes unrecoverably, any un-replicated writes are permanently lost even though the client received a success confirmation.

In **semi-synchronous replication**, the leader waits for acknowledgment from ==one synchronous follower== before returning success, while ==the remaining followers replicate asynchronously==.

## Related
- [[Leader-Based Replication]]
- [[Replication]]
- [[Replication Lag]]
- [[Replication Log]]
- [[Eventual Consistency]]
- [[Availability]]
- [[CAP Theorem]]
