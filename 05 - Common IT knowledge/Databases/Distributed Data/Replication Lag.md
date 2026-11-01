---
aliases:
  - Asynchronous Replication Lag
---
The delay between a write happening on the leader and being reflected on a follower. May be only a fraction of a second, and not noticeable in practice

Usually it is the main cause of the [[Eventual Consistency]].

## Problems with Replication Lag

**Read your own writes**. With asynchronous replication, there is a problem: if the user views the data shortly after making a write, the new data may not yet have reached the replica. To the user, it looks as though the data they submitted was lost, so they will be understandably unhappy. In this case we would need [[Consistency Levels#A. Read-Your-Writes Consistency]]. This is a guarantee that if the user reloads the page, they will always see any updates they submitted themeselves. This is even harder to handle in case of multiple devices - each of them might route to a different data center.

**Monotonic Reads**: User might see things go back in time. 