Multiple leaders ([[Active-active systems|active-active replication]]) means writes are accepted by more than one node, typically across multiple datacenters. This lowers local write latency and allows scaling writes across regions.

The biggest problem with multi-leader replication is that **write conflicts** can occur, which means conflict resolution is required.

Use cases for multi-leader replication:
1. **Multi-datacenter operation** — With a normal [[Leader-Based Replication]] setup, the leader lives in one datacenter and all writes must cross the WAN to reach it. In a multi-leader configuration, you have a leader in each datacenter. Within each datacenter, regular leader-follower replication is used; between datacenters, each datacenter’s leader replicates its changes asynchronously to the leaders in other datacenters.
2. **Clients with offline operation** — mobile calendar syncs, notes apps, etc. In this design, every device acts as a local leader with its own database that syncs asynchronously with other devices when online. CouchDB is designed for this model.
3. **Collaborative editing** — To guarantee no editing conflicts, an app would have to lock a document before editing (equivalent to single-leader replication). For real-time collaboration (keystroke-level edits without locking), each user edits a local replica and conflicts are resolved continuously.

## Handling write conflict

In a single-leader database, the second writer will either block and wait for the first write to complete, or abort the second write transaction, forcing the user to retry. In a multi-leader setup, **both writes succeed locally**, and the conflict is only detected asynchronously later when changes replicate.

**Conflict avoidance**. The simplest strategy for dealing with conflicts is to avoid them: if the application routes all writes for a particular user/record to the same datacenter leader, conflicts cannot occur (until a datacenter fails over and forces re-routing). Since many implementations of multi-leader replication handle conflicts poorly, avoiding conflicts is frequently recommended.

**Converging towards a consistent state**. If each replica simply applied writes in the order it saw them, replicas would permanently diverge because network arrival order is not guaranteed across datacenters. Thus, the database must resolve conflicts in a *convergent way*, guaranteeing all replicas arrive at the same final value:
- **Last Write Wins (LWW)**: Assign each write a unique ID/timestamp and keep the highest — simple, but **prone to silent data loss** (especially under clock skew).
- **Replica priority**: Give each replica a unique ID and always prefer writes from the higher-numbered replica — also loses data.
- **Merge values**: Concatenate or union conflicting values (e.g., `B/C`).
- **Preserve conflict**: Record both conflicting versions in an explicit data structure and resolve later (in app code or via user prompt).

**Custom conflict resolution logic**. Most multi-leader tools let you plug in application code to resolve conflicts:
1. **On write** — as soon as the DB detects a conflict in the replication stream, it invokes a background handler (cannot prompt the user).
2. **On read** — the DB stores all conflicting versions (siblings) and returns them to the application on the next read, which resolves them (automatically or by prompting the user) and writes the result back.

**What is a conflict?** Some conflicts are obvious — two leaders updating the same field of the same row. Others are subtle: in a meeting room booking system, two leaders can insert *different* booking rows for the same room at overlapping times. Neither row conflicts on primary key, yet the business invariant ("no double bookings") is violated (analogous to [[Write Skew]]).

### Automatic conflict resolution

Conflict resolution rules quickly become complicated, and custom code is error-prone. Amazon's shopping cart famously resurrected removed items when sibling merges only took the union of additions.

- **[[CRDT|Conflict-free Replicated Data Types (CRDTs)]]** — data structures (sets, maps, ordered lists, counters) that can be concurrently edited and automatically converge via commutative/associative/idempotent two-way merges (used in Riak 2.0, Redis Enterprise).
- **Mergeable persistent data structures** — track history explicitly like Git and use a three-way merge function.
- **Operational Transformation (OT)** — conflict resolution algorithm behind collaborative editors like Google Docs for concurrent edits to ordered character lists.

## Single vs Multi-leader replication

In multi-leader replication each datacenter has a local leader, so write latency is much lower and each datacenter can continue accepting writes during inter-datacenter network outages (unlike single-leader, where WAN outages block writes from remote regions or trigger failover).

## Multi-Leader Replication Topologies

A **replication topology** describes the communication paths along which writes propagate from one leader to another (with 2 leaders there is only one topology; with 3+ leaders there are several):

- **All-to-all**: Every leader sends its writes directly to every other leader. Most fault-tolerant, but some network links may be faster than others, causing **causality violations** (an `UPDATE` arriving before the `INSERT` it modifies — requires version vectors to order).
- **Circular (ring)**: Each node forwards writes to one neighbor (MySQL's historical default).
- **Star (or tree)**: One designated root node forwards writes to all other nodes.

In circular and star topologies, each write is tagged with the IDs of all nodes it has passed through to prevent infinite replication loops. However, both circular and star topologies have a **single point of failure**: one dead node interrupts replication across the cluster until reconfigured.

> [!warning] Failure modes & trade-offs
> Multi-leader replication is often bolted onto databases as an afterthought and has subtle traps: **auto-incrementing keys**, **triggers**, and **integrity constraints** (unique email, foreign keys) break easily across independent leaders. Avoid multi-leader replication inside a single datacenter — the operational complexity almost never pays off there.

## Why is conflict avoidance often preferred over automatic conflict resolution in multi-leader setups? #card
Conflict resolution is notoriously subtle and error-prone (LWW silently drops data, custom handlers introduce bugs, and cross-record invariants like double-booking cannot be resolved automatically). Routing all writes for a given user or entity to the same datacenter leader eliminates conflicts in normal operation.

## What is a key failure mode of an all-to-all multi-leader topology vs star/circular topologies? #card
- **Star / Circular**: A single node failure breaks the replication path for other nodes.
- **All-to-all**: Avoids single points of failure, but varying network delays can deliver causally dependent writes out of order (e.g., an `UPDATE` arrives before its `INSERT`), requiring version vectors to preserve causality.

In [[Multi-Leader Replication]], both conflicting writes succeed locally and the conflict is only detected ==asynchronously later during replication==, whereas single-leader replication blocks or aborts the second writer immediately.

Never deploy multi-leader replication within a single datacenter; use it only for multi-region write latency, offline clients, or collaborative editing. #card-reminder

## Related
- [[Leader-Based Replication]]
- [[Leaderless Replication]]
- [[CRDT]]
- [[Active-active systems]]
- [[Replication]]
- [[Eventual Consistency]]
- [[Types of Writes in distributed systems]]
- [[Write Skew]]
