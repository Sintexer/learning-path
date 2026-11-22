---
flashcards:
  q-s99j: { nid: 1795258118308, hash: qeqvj48f, sync: ct8qvueq }
  q-j4fv: { nid: 1795261497664, hash: ct896a3t, sync: nd5nwwuv }
  q-iqjh: { nid: 1795356725447, hash: kuwefhz5, sync: e56yq6m7 }
  q-r8hr: { nid: 1795356725466, hash: 8uj4srtk, sync: f8xq7egy }
---
In a leaderless configuration, failover does not exist — there is no need to choose a new leader when a node fails.

The client sends the write to all replicas in parallel; available replicas accept the write while an unavailable replica misses it. When reading, the client queries multiple replicas and uses version numbers to pick the newest value.

After an unavailable node comes back online, it catches up via two mechanisms:
1. **[[Read Repair]]** — the client detects a stale value on read and writes the newer value back to the stale replica.
2. **[[Anti-Entropy]]** — a background process continuously compares replicas and copies missing data (order is not guaranteed and lag can be significant).

Lack of an [[Anti-Entropy]] process reduces durability: rarely read keys may never be repaired on recovering nodes.

## Quorums for reading and writing

If every successful write is confirmed by at least 2 of 3 replicas, at most 1 replica can be stale. Reading from at least 2 replicas guarantees at least one up-to-date response, even if the third replica is down or slow.

More generally, with $n$ replicas, $w$ write acknowledgments, and $r$ read responses, as long as ==w + r > n==, we expect to read an up-to-date value because at least one of the $r$ replicas overlaps with the $w$ replicas. Reads and writes obeying this condition are called **[[Quorum Consensus|quorum reads and writes]]**.
^q-iqjh

Reads and writes are still sent to all $n$ replicas in parallel; $w$ and $r$ only control how many acknowledgments the client waits for before succeeding.

> [!info] What is n
> In real clusters (Amazon Dynamo, [[Apache Cassandra]], Riak), a cluster might have **100 nodes** while your replication factor ($n$) is only **3**. **[[Consistent Hashing]]** maps each key (e.g., `User_123`) to 3 specific **Home Nodes** (e.g., Nodes A, B, and C); the remaining nodes do not store that key.

## Stale values read cases #card

1. If a [[Sloppy Quorum]] is used
2. If two writes happen simultaneously - conflict resolution mechanism might overwrite one variant 
3. If a write happens concurrently to the read - some replicas might return old values, some return updated
4. If a write succeeded on some replicas and failed on others
5. If a node carrying a new value fails and its data is restored from a node holding a stale value, the number of up-to-date replicas might fall bellow the required threshold.
6. Even if everything works correctly, an unlucky timing may cause errors.
^q-s99j

## Multi-datacenter operation

[[Apache Cassandra]] and Voldemort include nodes across all datacenters in $n$, configured explicitly *per datacenter* (total $n$ is the sum). Clients typically wait only for **`LOCAL_QUORUM`** inside their local datacenter (~2 ms) while cross-DC writes replicate asynchronously in the background (~100 ms WAN link).

- **Best practice**: Put at least **3 replicas in every datacenter** serving local traffic (`{DC1: 3, DC2: 3}`), not `{DC1: 3, DC2: 1}` — a single replica in `DC2` cannot sustain `LOCAL_QUORUM` during maintenance or failure.
- Riak keeps client/node quorums strictly inside one datacenter and replicates cross-DC asynchronously in the background (similar to [[Multi-Leader Replication]]).

## Detecting Concurrent Writes

Because Dynamo-style databases let multiple clients write concurrently to the same key (even under strict quorums, or during [[Read Repair]] and [[Sloppy Quorum#Hinted handoff|hinted handoff]]), replicas must resolve conflicts to converge.

See **[[Version Vectors and Concurrent Write Resolution]]** for full details on happens-before causality, sibling merging, tombstones, and Dotted Version Vectors.

- **Last Write Wins (LWW)**: LWW achieves the goal of eventual convergence, but at the cost of ==durability==: if there are several concurrent writes to the same key, even if they were all reported as successful to the client (because they were written to w replicas), only one of the writes will survive and the others will be silently discarded.
^q-j4fv

## Monitoring

In leaderless systems there is no leader write-position to subtract from a follower's position to measure [[Replication Lag|replication lag]]. If a cluster relies only on [[Read Repair]] without [[Anti-Entropy]], there is no upper bound on how stale a rarely-read value can become — monitoring data freshness is critical.

> [!warning] Failure modes & trade-offs
> - **Quorums ($w + r > n$) do NOT guarantee [[Linearizability]]**: Concurrent reads during an in-flight write can observe the new value on one read and the old value on a subsequent read unless synchronous read-repair runs before returning.
> - **Clock skew under LWW**: [[Apache Cassandra]]'s default LWW relies on wall-clock timestamps; a node whose clock runs ahead silently overwrites newer writes.

## Why is a single version number insufficient in leaderless replication, requiring a Version Vector instead? #card
With multiple replicas independently accepting concurrent writes, a single scalar counter cannot distinguish whether write B overwrote write A or happened concurrently with it. A **version vector** maintains one counter per replica per key, letting replicas identify which values causally supersede others and which must be kept as concurrent siblings.
^q-r8hr

## Related
- [[Version Vectors and Concurrent Write Resolution]]
- [[Sloppy Quorum]]
- [[Read Repair]]
- [[Anti-Entropy]]
- [[CRDT]]
- [[Types of Writes in distributed systems]]
- [[Consistent Hashing]]
- [[Quorum Consensus]]
- [[Linearizability]]
- [[Apache Cassandra]]
- [[Eventual Consistency]]
