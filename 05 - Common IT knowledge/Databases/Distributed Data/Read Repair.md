In [[Leaderless Replication|leaderless (Dynamo-style)]] databases (such as [[Apache Cassandra]] and Riak), **read repair** is a mechanism where a client (or coordinator node) detects that a replica returned a stale value during a quorum read and asynchronously writes the newer version back to that stale replica.

## How It Works

1. A client reads key `K` from $r$ replicas in parallel (e.g., `Node A` returns `v7`, `Node B` returns `v7`, `Node C` returns `v6`).
2. Comparing version numbers (or timestamps / version vectors), the coordinator determines `v7` is newer and returns `v7` to the client.
3. In the background (or synchronously, depending on configuration), the coordinator sends `v7` to `Node C` to bring it up to date.

## Where Read Repair Works Well — and Where It Fails

- **Hot keys**: Frequently read keys converge quickly without waiting for scheduled [[Anti-Entropy]] runs.
- **Cold keys**: Keys that are rarely read are *never* healed by read repair. If a database relies solely on read repair without [[Anti-Entropy]], a key may silently drop below its replication factor and be lost if the surviving up-to-date replica fails.

> [!warning] Failure modes & trade-offs
> - **Read amplification & coordinator overhead**: Blocking read repair adds write latency onto the read path; non-blocking (probabilistic) read repair lowers read latency but delays convergence.
> - **Conflicts during repair**: If two concurrent writes occurred with different values (neither causally dominating the other), read repair must apply the conflict resolution policy (e.g., LWW, sibling merging, or [[CRDT]] merge) — otherwise LWW can silently drop one of the writes.

Read repair:::A [[Leaderless Replication]] mechanism where a read query from multiple replicas detects a stale replica and writes the newer value back to it.

## Why is read repair alone insufficient for durability in a leaderless database? #card
Read repair only updates replicas for keys that clients actually read. Rarely read ("cold") keys may never be repaired after a node outage, leaving fewer than $n$ up-to-date copies and risking permanent data loss if the remaining healthy replica fails. It must be paired with [[Anti-Entropy]].

## Related
- [[Anti-Entropy]]
- [[Leaderless Replication]]
- [[Sloppy Quorum]]
- [[Eventual Consistency]]
- [[Apache Cassandra]]
