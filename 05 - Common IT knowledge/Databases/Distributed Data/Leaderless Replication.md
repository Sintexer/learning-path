---
flashcards:
  q-s99j: { nid: 1795258118308, hash: f25rd6hd, sync: vvav2nny }
  q-j4fv: { nid: 1795261497664, hash: sbm6cejq, sync: u5sx8xh2 }
---
In a leaderless configuration, failover does not exist - there is no need to choose a new leader in case a node fails.`

The client sends the write to all replicas in parallel, and the other available replicas accept the write but the unavailable replica misses it. When client reads, it reads from multiple replicas, to avoid reading a stale value from a turned back online replica. Version numbers are used to determine which value is newer.

After an unavailable node comes back online, how does it catch up on the writes that it missed? Two mechanisms:
1. *Read repair* - client detects a stale value read from a replica and updates the up-to-date value back to the replica.
2. *Anti-entropy process* - a background process that constantly looks for differences in the data between replicas and copies any missing data from one replica to another. Order is usually no guaranteed, and there may be a significant delay before data is copied.

Not all systems implement both of these. Lack of the anti-entropy process implies reduced durability - a rarely read value is not guaranteed to be present on most nodes.

## Quorums for reading and writing

If we know that every successful write is guaranteed to be present on at least two out of three replicas, that means at most one replica can be stale. Thus, if we read from at least two replicas, we can be sure that at least one of the two is up to date. If the third replica is down or slow to respond, reads can nevertheless continue returning an up-to-date value.

More generally, if there are n replicas, every write must be confirmed by w nodes to be considered successful, and we must query at least r nodes for each read. (In our example, n = 3, w = 2, r = 2.) As long as w + r > n, we expect to get an up-to-date value when reading, because at least one of the r nodes we’re reading from must be up to date. Reads and writes that obey these r and w values are called **quorum reads and writes**.

Normally, reads and writes are always sent to all n replicas in parallel. The parameters w and r determine how many nodes we wait for - i.e., how many of the n nodes need to report success before we consider the read or write to be successful.

> [!info] What is n
> In textbook examples of quorums (w+r>n), we usually pretend the database only has n nodes in total (e.g., n=3), and _every_ node holds _all_ the data. If that were true in the real world, you would be right: you could write to any node at any time.
> 
> However, in the real world (like in Amazon Dynamo, Cassandra, or Riak), **a cluster might have 100 nodes, while your replication factor (n) is only 3.**

### What are "Home" Nodes?

Imagine you have a cluster of **100 nodes**, and your Replication Factor (n) is **3**.  
When you write a piece of data (say, `User_123`), you do not write it to all 100 nodes. You only write it to 3 nodes.

How does the database know _which_ 3 nodes out of the 100 should hold `User_123`? It uses **Consistent Hashing**.  
The database hashes the key `User_123`, and the result points to 3 specific nodes (for example, **Node A, Node B, and Node C**).

**Nodes A, B, and C are the "Home Nodes" (or designated nodes) for `User_123`.**  
Nodes D through Z have no idea `User_123` exists. They are busy holding other data.

## Stale values read cases #card

1. If a [[Sloppy Quorum]] is used
2. If two writes happen simultaneously - conflict resolution mechanism might overwrite one variant 
3. If a write happens concurrently to the read - some replicas might return old values, some return updated
4. If a write succeeded on some replicas and failed on others
5. If a node carrying a new value fails and its data is restored from a node holding a stale value, the number of up-to-date replicas might fall bellow the required threshold.
6. Even if everything works correctly, an unlucky timing may cause errors.
^q-s99j

## Multi-datacenter operation

Cassandra and Voldemort implement their multi-datacenter support within the normal leaderless model: the number of replicas n includes nodes in all datacenters, and in the configuration you can specify how many of the n replicas you want to have in each datacenter. 

> [!tip] Fact
> In a multi-datacenter (Multi-DC) setup, you **do not** just define a single global number like n=5 and let the database randomly guess which datacenters get the data. Instead, **you explicitly configure n per datacenter**, and the total n is simply the sum of those numbers.

You _can_ configure `{DC1: 3, DC2: 1, DC3: 1}`, **but it is usually a bad idea.** Here is whyn:

> _"...the client usually only waits for acknowledgment from a quorum of nodes within its local datacenter so that it is unaffected by delays..."_

Sending data across the ocean (e.g., US to Europe) takes ~100ms. Sending data inside one datacenter takes ~1ms. You do not want your users waiting for cross-ocean network traffic on every write.

To solve this, engineers use *Local Quorum* instead of a global quorum.

- **Global Quorum:** n=6, so w=4. (Requires waiting for the ocean link).
- **Local Quorum:** The client connects to `US-East`. `US-East` has 3 replicas. A quorum of 3 is **2**.

When a user in New York updates their profile:

1. The client sends the write to a node in `US-East`.
2. That node sends the write to the **3 Home Nodes in US-East**, AND fires it across the internet to the **3 Home Nodes in US-West**.
3. As soon as **2 nodes in US-East** say "Success!", the database tells the user "Success!" (Takes ~2 milliseconds).
4. The 3 nodes in `US-West` receive the data a split-second later in the background.

**Why `{DC1: 3, DC2: 1, DC3: 1}` fails here:**  
If `DC2` only has 1 replica, you cannot do a quorum in `DC2` with fault tolerance! If that 1 node goes down for maintenance, users connected to `DC2` can no longer use `LOCAL_QUORUM`. Therefore, best practice is to put **3 replicas in every datacenter** that serves local traffic.

Riak keeps all communication between clients and database nodes local to one datacenter, so n describes the number of replicas within one datacenter. Cross-datacenter replication between database clusters happens asynchronously in the background, in a style that is similar to multi-leader replication

## Detecting Concurrent Writes

Dynamo-style databases allow several clients to concurrently write to the same key, which means the conflict may occur even if a strict quorum is used. This is similar to the situation of [[Multi-Leader Replication#Handling write conflict]]. Although in dynamo-style databases conflicts might occur during read repair or [[Sloppy Quorum#Hinted handoff|hinted handoff]] - events may arrive in different order to at different nodes.

In order to become eventually consistent, the replicas should converge toward the same value. How do they do that? One might hope that replicated databases would handle this automatically, but unfortunately most implementations are quite poor.

### Last wire wins

As long as we have some way of unambiguously determining which write is more “recent,” and every write is eventually copied to every replica, the replicas will eventually converge to the same value. Commonly abbreviated as **LWW**.

LWW achieves the goal of eventual convergence, but at the cost of ==durability==: if there are several concurrent writes to the same key, even if they were all reported as successful to the client (because they were written to w replicas), only one of the writes will survive and the others will be silently discarded.
^q-j4fv

The only safe way of using a database with LWW is to ensure that a key is only written once and thereafter treated as immutable, thus avoiding any concurrent updates to the same key. For example, a recommended way of using Cassandra is to use a UUID as the key, thus giving each write operation a unique key

### The "happens-before" relation and concurrency

Related to [[Types of Writes in distributed systems]].

If one operation happened before another, the later operation should overwrite the earlier operation, but if the operations are concurrent, we have a conflict that needs to be resolved.

To solve the concurrent writes, a version field is used:

- The server maintains a version number for every key, increments the version number every time that key is written, and stores the new version number along with the value written.
- When a client reads a key, the server returns all values that have not been overwritten, as well as the latest version number. A client must read a key before writing.
- When a client writes a key, it must include the version number from the prior read, and it must merge together all values that it received in the prior read. (The response from a write request can be like a read, returning all current values, which allows us to chain several writes like in the shopping cart example.)
- When the server receives a write with a particular version number, it can overwrite all values with that version number or below (since it knows that they have been merged into the new value), but it must keep all values with a higher version number (because those values are concurrent with the incoming write).

When a write includes the version number from a prior read, that tells us which previous state the write is based on. If you make a write without including a version number, it is concurrent with all other writes, so it will not overwrite anything — it will just be returned as one of the values on subsequent reads.

### Merging concurrently written values

A reasonable approach to merging siblings is to just take the union. If the two final siblings are `[milk, flour, eggs, bacon]` and `[eggs, milk, ham]`; note that milk and eggs appear in both, even though they were each only written once. The merged value might be something like `[milk, flour, eggs, bacon, ham]`, without duplicates.

However, if you want to allow people to also remove things from their carts, and not just add things, then taking the union of siblings may not yield the right result: if you merge two sibling carts and an item has been removed in only one of them, then the removed item will reappear in the union of the siblings. To prevent this problem, an item cannot simply be deleted from the database when it is removed; instead, the system must leave a marker with an appropriate version number to indicate that the item has been removed when merging siblings. Such a deletion marker is known as a **tombstone**. 

As merging siblings in application code is complex and error-prone, there are some efforts to design data structures that can perform this merging automatically: E.g. Riak and [[CRDT]].

### Version Vector

How does the [[#Merging concurrently written values]] change when there are multiple replicas, but no leader?

Single stored version value is not sufficient when there are multiple replicas accepting writes concurrently. Instead, we need to use a version number per replica as well as per key. Each replica increments its own version number when processing a write, and also keeps track of the version numbers it has seen from each of the other replicas. This information indicates which values to overwrite and which values to keep as siblings.

The collection of version numbers from all the replicas is called a version vector. A few variants of this idea are in use, but the most interesting is probably the dotted version vector, which is used in Riak 2.0. Version vectors are sent from the database replicas to clients when values are read, and need to be sent back to the database when a value is subsequently written. (Riak encodes the version vector as a string that it calls *causal context*.) The version vector allows the database to distinguish between overwrites and concurrent writes.

## Monitoring

Such Dynamo-style databases it is important to set-up monitoring to detect the stale data. If the freshness of the data falls behind significantly, it should alert ops to investigate the root- cause.

By subtracting a follower's current position from the leader's current position we can measure the replication lag. However in leaderless systems there is no specific order in which writes should be applied, and if there is no [[Anti-Entropy]] process and only the [[Read Repair]] is used, there is no limit on how stale a value may be.