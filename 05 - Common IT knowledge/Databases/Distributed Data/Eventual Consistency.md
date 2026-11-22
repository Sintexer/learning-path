**Eventual consistency** is a property of a distributed system (the weakest of the [[Consistency Levels]]) that guarantees that, if no new updates are made, all replicas will eventually converge to the same value.

While **[[Replication Lag|asynchronous replication lag]]** (the time it takes for data to physically travel from a primary node to secondary nodes over a network) is a primary driver, several other architectural factors and mechanisms cause eventual consistency:
1. **Multi-Leader and Leaderless Architectures (Concurrent Writes)**: [[Active-active systems]], [[Multi-Leader Replication]], or [[Leaderless Replication]]. Divergence arises not only from network lag, but because the system must resolve *mathematical and logical divergence* across concurrent writers (using LWW, version vectors, or [[CRDT]]s).
2. **Caching Layers (Application Level)**: Time-to-Live (TTL) caches (e.g., [[Redis]], [[Memcached]]) and cache-aside invalidation delays. If the message to invalidate or update the cache is dropped, delayed, or processed asynchronously via an event bus, clients reading from the cache experience eventual consistency.
3. **Asynchronous Messaging and [[Event-driven architecture|Event-Driven Architectures]]**: Publishing updates via [[Kafka]] or [[Outbox Pattern]] / [[Change Data Capture]] to downstream read models ([[CQRS]]).
4. **Network Partitions and Node Failures**: Under [[CAP Theorem]], systems that prioritize availability during a partition allow replicas to diverge temporarily.
5. **Gossip Protocols**: In fully decentralized peer-to-peer systems, nodes have no central coordinator. Instead, they periodically "gossip" with random neighbors to share state, taking multiple rounds of probabilistic communication for updates to permeate the cluster.
6. **[[Read Repair]] and [[Anti-Entropy]] Processes**: Background and on-read convergence mechanisms in Dynamo-style databases.
7. **Disconnected or Offline-First Clients**: A mobile app writing locally on an airplane or subway is fundamentally eventually consistent; convergence depends on when the user reconnects.

> [!warning] Failure modes & trade-offs
> Eventual consistency says **nothing** about *when* replicas will converge or *what* anomalies clients see in the meantime (lost updates, reading your own write disappearing, time moving backward). Always specify whether you also need client-centric guarantees ([[Consistency Levels#4. Client-Centric Consistency Levels|Read-Your-Writes, Monotonic Reads]]) or conflict-free merging ([[CRDT]]).

Eventual consistency:::A weak consistency guarantee stating that if no new updates are made, all replicas will eventually converge to the same value — with no bound on how long convergence takes.

## Name four architectural causes of eventual consistency beyond database replication lag. #card
1. Application-level caches with TTL or async invalidation ([[Redis]], [[Memcached]]).
2. Asynchronous messaging / [[Event-driven architecture]] / [[CQRS]] projections.
3. [[Multi-Leader Replication]] or [[Leaderless Replication]] concurrent writes awaiting conflict resolution.
4. Gossip protocols or offline-first mobile clients syncing on reconnect.

Never treat "eventual consistency" as an excuse for silent data loss: pair it with explicit conflict resolution (version vectors / [[CRDT]]) and [[Reconciliation]] checks for critical business invariants. #card-reminder

## Related
- [[Consistency Levels]]
- [[Replication Lag]]
- [[CAP Theorem]]
- [[Read Repair]]
- [[Anti-Entropy]]
- [[CRDT]]
- [[Leaderless Replication]]
- [[Multi-Leader Replication]]
- [[CQRS]]
