Multiple leaders means the writes are accepted by multiple data centers. This is a way to scale the writes horizontally.

The biggest problem with multi-leader replication is that write conflicts can occur, which means that conflict resolution is required.

Use cases for multi-leader replication:
1. **Multi-datacenter operation** - With a normal [[Leader-Based Replication]] setup, the leader has to be in one of the datacenters, and all writes must go through that datacenter. In a multi-leader configuration, you can have a leader in each datacenter. Within each datacenter, regular leader-follower replication is used; between datacenters, each datacenter’s leader replicates its changes to the leaders in other datacenters.
2. **Clients with offline operation** - mobile calendar syncs, etc. In this design, any mobile phone is basically a separate leader that should be synced with others (each device is a datacenter). CouchDB is designed for this kind of operation.
3. **Collaborative editing** - If you want to guarantee that there will be no editing conflicts, the application must obtain a lock on the document before a user can edit it. If another user wants to edit the same document, they first have to wait until the first user has committed their changes and released the lock. This collaboration model is equivalent to single-leader replication with transactions on the leader. However, for faster collaboration, you may want to make the unit of change very small (e.g., a single keystroke) and avoid locking. This approach allows multiple users to edit simultaneously, but it also brings all the challenges of multi-leader replication, including requiring conflict resolution.

## Handling write conflict

In a single-leader database, the second writer will either block and wait for the first write to complete, or abort the second write transaction, forcing the user to retry the write. On the other hand, in a multi-leader setup, both writes are successful, and the conflict is only detected asynchronously at some later point in time.

**Conflict avoidance**. The simplest strategy for dealing with conflicts is to avoid them: if the application can ensure that all writes for a particular record go through the same leader, then conflicts cannot occur. Since many implementations of multi-leader replication handle conflicts quite poorly, avoiding conflicts is a frequently recommended approach

**Converging towards a consistent state**: If each replica simply applied writes in the order that it saw the writes, the database might end up in an inconsistent state - the order is not guaranteed in a multi-datacenter operation. Thus, the database must resolve the conflict in a *convergent way*, which means that all replicas must arrive at the same final value when all changes have been replicated. This can be achieved by ordering writes by some id, timestamp, key and skip "outdated" updates. Another way is to pick writes based on which replica id is higher. Third way is to merge values together - e.g. concatenate titles and let user de-merge it manually. Last option is to store conflict until the user resolves it manually. 

**Custom conflict resolution logic**. As the most appropriate way of resolving a conflict may depend on the application, most multi-leader replication tools let you write conflict resolution logic using application code. That code may be executed on write or on read: 
1. On write - on write conflict DB calls an application code handler
2. On read - DB stores the conflic in a special data structure and prompts user to resolve the conflict on read, or resolve the conflict automatically. 

**What is a conflict?** Some conflicts are obvious - modification of the same entity field is likely a conflict. But some kinds of conflict can be more subtle to detect. For example, consider a meeting room booking system: it tracks which room is booked by which group of people at which time. This application needs to ensure that each room is only booked by one group of people at any one time (i.e., there must not be any overlapping bookings for the same room). In this case, a conflict may arise if two different bookings are created for the same room at the same time.
### Automatic conflict resolution

Conflict resolution rules can quickly become complicated, and custom code can be error-prone. Amazon is a frequently cited example of surprising effects due to a conflict resolution handler: for some time, the conflict resolution logic on the shopping cart would preserve items added to the cart, but not items removed from the cart.

- Conflict-free replicated datatypes (CRDTs) are a family of data structures for sets, maps, ordered lists, counters, etc. that can be concurrently edited by multiple users, and which automatically resolve conflicts in sensible ways. Some CRDTs have been implemented in Riak 2.0.
- Mergeable persistent data structures track history explicitly, similarly to the Git version control system, and use a three-way merge function (whereas CRDTs use two-way merges).
- Operational transformation is the conflict resolution algorithm behind collaborative editing applications such as Etherpad and Google Docs. It was designed particularly for concurrent editing of an ordered list of items, such as the list of characters that constitute a text document.
## Single vs Multi-leader replication

In multi-leader replication each data center has a leader, so the performance is better. In case of the data center outage, other datacenters would continue normal operation (in single-leader design, new leader might be elected in a different datacenter). And the multi-leader replication is less dependent on the public internet stability. 

## Multi-Leader Replication Topologies

If you have only 2 leaders there could be only one possible topology - replication topology is a communication paths along which writes are propagated from one node to another. With more than 2 leaders various topologies are possible.

The most general topology is **all-to-all**, in which every leader sends its writes to every other leader. However, more restricted topologies are also used: for example, MySQL by default supports only a **circular** topology, in which each node receives writes from one node and forwards those writes (plus any writes of its own) to one other node. Another popular topology has the shape of a **star**: one designated root node forwards writes to all of the other nodes. The star topology can be generalized to a tree.

In circular and star topologies, a write may need to pass through several nodes before it reaches all replicas. Therefore, nodes need to forward data changes they receive from other nodes. To prevent infinite replication loops, each node is given a unique identifier, and in the replication log, each write is tagged with the identifiers of all the nodes it has passed through.

Star and circular topologies are much more prone to interrupt of the flow of replication messages between replica if even only one node fails.