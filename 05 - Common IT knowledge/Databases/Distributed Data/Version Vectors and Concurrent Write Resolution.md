---
flashcards:
  q-wvwj: { nid: 1795361722080, hash: dxbhvpbk, sync: znwbx2jg }
---

In [[Leaderless Replication]] and [[Multi-Leader Replication]], multiple clients or replicas can accept writes to the same key concurrently. Replicas must detect which writes causally supersede earlier writes and which are **concurrent conflicts** (see [[Types of Writes in distributed systems]]) so all nodes eventually converge.

## 1. Last Write Wins (LWW)

Assign every write a timestamp (or monotonic ID) and keep only the highest value, discarding all older ones.

- **Pros**: Simple and guarantees eventual convergence without sibling bookkeeping.
- **Cons**: **Silent data loss** — concurrent writes reported as successful to clients are silently dropped, and clock skew across nodes can let an older write overwrite a newer one.
- **Safe usage**: Only safe when keys are **immutable** (written once with a unique UUID key, never updated in place).

## 2. Single-Replica Causal Versioning ("Happens-Before")

With one server tracking causality per key:
1. The server stores a version number per key and increments it on every write.
2. Clients **read before writing**; the server returns all non-overwritten values (siblings) plus the current version number.
3. On write, the client sends back the version number from its prior read along with the merged value.
4. The server overwrites all values at or below that version number, and **keeps values with a higher version number** as concurrent siblings.

A write sent *without* a version number is concurrent with everything and overwrites nothing.

## 3. Merging Siblings & Tombstones

When a client reads multiple concurrent siblings (e.g., shopping carts `[milk, flour, eggs]` and `[eggs, milk, ham]`):
- **Union merge** works for additions (`[milk, flour, eggs, ham]`), but **resurrects deleted items** if one sibling removed an item that still exists in the other.
- **Tombstones**: Deletions cannot simply erase a record; the system writes a versioned deletion marker (**tombstone**) so sibling merges know the item was removed.
- Because application-level sibling merging is error-prone, systems increasingly use [[CRDT|CRDTs]] for automatic conflict-free convergence.

## 4. Version Vectors (Multiple Replicas)

A single scalar version number fails when multiple replicas accept writes independently without a leader. Instead, each key carries a **version vector** — a map of `(replica_id -> counter)`:

- Each replica increments its own counter when processing a write and tracks the highest counter it has seen from every other replica.
- Replicas return the version vector to clients on read (Riak calls this encoded vector the **causal context**); clients echo it back on write.
- Comparing two version vectors cleanly distinguishes **causal overwrite** (every entry in A ≤ B) from **concurrent conflict** (A has a higher counter for one replica, B has a higher counter for another — keep both as siblings).
- Modern Dynamo-style systems (e.g., Riak 2.0) use **Dotted Version Vectors** to avoid false conflicts when multiple clients write through the same coordinator replica.

> [!warning] Failure modes & trade-offs
> - **Sibling explosion**: If clients repeatedly write without passing back the causal context (version vector), siblings accumulate unboundedly on every write, bloating record size and blowing up read latency.
> - **Tombstone accumulation**: Tombstones cannot be garbage-collected until all replicas have seen them (e.g., Cassandra's `gc_grace_seconds`); purging too early resurrects deleted data during [[Anti-Entropy]] repair.

## Why does taking the union of concurrent siblings fail when deletes are allowed, and how is it solved? #card
If one sibling removes an item while another sibling still contains it, a plain union resurrects the deleted item. Systems solve this by writing a versioned deletion marker called a **tombstone** instead of physically erasing the item immediately, so merges respect the deletion.

Version Vector ::: A per-key map of version counters (one per replica) that lets leaderless replicas distinguish causal overwrites from concurrent writes.

When using Last Write Wins (LWW) in a distributed database, the only way to avoid silent data loss is to treat each key as ==immutable (write-once, e.g., using a unique UUID per write)==.
^q-wvwj

## Related
- [[Leaderless Replication]]
- [[Multi-Leader Replication]]
- [[Types of Writes in distributed systems]]
- [[CRDT]]
- [[Anti-Entropy]]
- [[Read Repair]]
- [[Eventual Consistency]]
