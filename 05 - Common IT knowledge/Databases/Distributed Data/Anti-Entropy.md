---
flashcards:
  q-zkf5: { nid: 1795356725893, hash: 57kxsfey, sync: zivcjeir }
  q-vtat: { nid: 1795356725916, hash: rqvbe56k, sync: w3i3d3zq }
---

In [[Leaderless Replication|leaderless (Dynamo-style)]] databases (such as [[Apache Cassandra]] and Riak), an **anti-entropy process** is a background job that periodically compares data across replicas and copies missing or outdated records so all replicas converge toward [[Eventual Consistency]].

## How Anti-Entropy Works

Comparing raw datasets across nodes over the network would consume massive bandwidth. Instead, production systems use **Merkle trees** (hash trees):
1. Each replica computes a hierarchical tree of cryptographic hashes over its key range (each leaf hashes a bucket of keys; each parent hashes its children).
2. Two replicas exchange only the **root hash**. If the roots match, the replicas are identical and the sync finishes immediately.
3. If the roots differ, they walk down only the differing subtrees until they pinpoint the exact mismatched key ranges, transferring only the missing or newer entries.

## Anti-Entropy vs [[Read Repair]]

- **[[Read Repair]]** only heals keys that clients actively read. Cold/rarely read keys can remain stale or under-replicated indefinitely.
- **Anti-entropy** scans *all* data in the background regardless of read traffic, protecting durability for cold data and recovering nodes that were offline longer than the [[Sloppy Quorum#Hinted handoff|hinted handoff]] window.

> [!warning] Failure modes & trade-offs
> - **Not real-time & unordered**: Unlike a [[Replication Log]], anti-entropy does not copy writes in any specific order and may run hours apart (e.g., `nodetool repair` in [[Apache Cassandra]]). Until it runs, replicas remain diverged.
> - **Operational I/O & CPU cost**: Building Merkle trees scans every [[SSTable]] on disk, causing heavy disk read and compaction load. In Cassandra, skipping periodic repair beyond `gc_grace_seconds` risks **zombie data resurrection** — deleted tombstones get purged on one replica before reaching another, causing deleted rows to reappear.

## Why do Dynamo-style databases use Merkle trees for anti-entropy? #card
Merkle trees let two replicas compare millions of keys by exchanging a single root hash first and recursively descending only into differing subtrees — minimizing network bandwidth to just the mismatched key ranges.
^q-zkf5

## What happens in Cassandra if anti-entropy repair does not run before `gc_grace_seconds` expires? #card
Tombstones (deletion markers) are purged from compacted [[SSTable]]s on replicas that saw the delete. If another replica missed the delete, future anti-entropy or [[Read Repair]] will copy the old live row back, **resurrecting deleted data** ("zombie data").

Unlike [[Read Repair]], ==anti-entropy== heals cold, rarely read keys in the background and is essential for long-term durability in [[Leaderless Replication]].
^q-vtat

## Related
- [[Read Repair]]
- [[Leaderless Replication]]
- [[Sloppy Quorum]]
- [[Eventual Consistency]]
- [[Apache Cassandra]]
- [[SSTable]]
