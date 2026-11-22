This mode of [[Replication]] (also known as *primary-replica* or *active-passive* replication; see [[Active-passive systems]]) is a built-in feature of most [[Relational Database|relational databases]] ([[PostgreSQL]], MySQL, Oracle) as well as many distributed stores ([[MongoDB]], [[Kafka]] partitions). Under the hood, the leader feeds the [[Replication Log]] to the followers.

## Leader

One of the replicas is designated the *leader* (primary). When clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage.

Only the leader accepts writes in this solution.

## Follower

Non-leader replicas receive the data from the leader whenever it writes new data to its local storage. This event stream is called a *[[Replication Log|replication log]]* or *change stream*. Followers apply writes in the exact same order as they were processed on the leader.

When a client wants to read from the database, it can query either the leader or any of the followers (scaling read throughput). However, writes are only accepted on the leader.

To spawn new followers without downtime, you capture a consistent database snapshot (most databases have this feature natively for backups), copy it to the new follower, and have the follower request all writes since the snapshot's exact position in the [[Replication Log|replication log]] (e.g., PostgreSQL LSN or MySQL binlog coordinates). After processing the backlog, the follower is caught up and processes live writes.

## Handling Node Outages

- **Follower failure (catch-up recovery)**: Each follower keeps a local log of the last transaction processed. On restart, it reconnects to the leader and requests all changes after that log position.
- **Leader failure (failover)**:
  1. Determine the leader has failed (typically via timeout, e.g., 30 seconds without heartbeat).
  2. Choose a new leader via election (consensus like [[Raft]]/[[Paxos]]) or an external controller — ideally the follower with the most up-to-date log.
  3. Reconfigure clients and remaining followers to route writes to the new leader.

> [!warning] Failure modes & trade-offs
> Automatic failover is notoriously treacherous in production:
> - **Committed write loss**: With [[Synchronous vs Asynchronous Replication|asynchronous replication]], the newly promoted leader may lack recent writes from the old leader. When the old leader rejoins, those un-replicated writes are typically discarded — which is catastrophic if external systems (like a [[Redis]] cache or another service) already observed them (e.g., GitHub's famous auto-increment primary key reuse incident).
> - **[[Split-Brain]]**: During a network partition, two nodes can both believe they are the leader and both accept writes, causing silent data corruption. Prevent with [[Quorum Consensus|quorum-based]] leader election and [[Fencing Token|fencing tokens]] (or STONITH — Shoot The Other Node In The Head).
> - **Timeout tuning trap**: Too short a failover timeout triggers unnecessary failovers during temporary load spikes or GC pauses, making outages worse; too long extends write downtime.

## What are the three major hazards of automatic leader failover? #card
1. **Data loss / conflicting IDs**: Under async replication, the new leader may lack recently acknowledged writes, which get discarded when the old leader rejoins.
2. **[[Split-Brain]]**: Two nodes both believe they are the leader during a network partition and accept conflicting writes.
3. **Hair-trigger failovers**: Too-short timeouts mistake temporary load spikes or GC pauses for leader death, destabilizing the cluster.

How does a new follower join a leader-based cluster without locking the database for writes?::Take a consistent snapshot at a known log position, copy it to the follower, then stream and apply all [[Replication Log]] entries since that exact position until caught up.

In [[Leader-Based Replication]], prevent a zombie old leader from corrupting storage after failover by using ==[[Fencing Token|fencing tokens]]== or quorum-enforced epochs.

## Related
- [[Replication]]
- [[Synchronous vs Asynchronous Replication]]
- [[Replication Log]]
- [[Replication Lag]]
- [[Multi-Leader Replication]]
- [[Leaderless Replication]]
- [[Split-Brain]]
- [[Fencing Token]]
- [[Raft]]
- [[Active-passive systems]]
