Replication means keeping the copy of the same data on different machines connected via the network. There are several reasons to replicate data: Latency, fault tolerance, horizontal scaling.

Each node that stores a copy of the database is called a replica. With multiple replicas a question inevitably arises: how do we ensure that all the data ends up on all the replicas?

The most common solution for this is called *[[Leader-Based Replication]]*.

An important detail of a replicated system is whether the replication happens [[Synchronous vs Asynchronous Replication|synchronously or asynchronously]].