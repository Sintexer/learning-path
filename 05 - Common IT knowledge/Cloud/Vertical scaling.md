**Increase the instance size (scale up).** If too many requests are sent to a single active-passive system, the active server will become unavailable and, hopefully, fail over to the passive server. But this doesn't solve the underlying capacity problem.

With [[Active-passive systems]], you need **vertical scaling**. This means increasing the size of the server (more CPU, RAM, or faster I/O). To vertically scale an active-passive system with minimal downtime, one must stop the backup instance, change its size, start the backup instance, reroute traffic to this instance, and then do the same to the previous primary instance—lots of manual work unless automated via [[IaaC]]. When the number of requests reduces, you must repeat the same operation. Even though there aren't that many steps involved, it's operational overhead and mainly practical in cloud/virtualized environments.

Another disadvantage is that a server can only scale vertically up to a certain hardware limit, and high-end instance sizes grow super-linearly in cost. When that limit is reached, the only option is to split the workload across multiple systems ([[Data partitioning]] or [[Microservices App|service decomposition]]). This can require massive application rewriting.

This is where the [[Active-active systems|active-active system]] can help. When there are too many requests, you can [[Horizontal scaling|scale this system horizontally]] by adding more servers.

> [!tip] Production reality
> Don't dismiss vertical scaling prematurely: for single-writer databases ([[PostgreSQL]]) or stateful monoliths, scaling up to a beefier node avoids distributed-systems failure modes ([[Replication Lag]], [[Split-Brain]], cross-shard joins) and buys months or years of engineering focus.

## Flashcards

Vertical scaling (scaling up):::Increasing the capacity (CPU, RAM, disk IOPS) of an existing single server rather than adding more servers.

What are the two main limitations of vertical scaling?::A hard hardware/cost ceiling per single machine and the need for restarts or failover to resize (making elastic scale-down awkward).

## How do you vertically scale an active-passive pair with minimal downtime? #card

Resize the passive standby first, verify it is healthy and synced, fail traffic over to the resized standby, then resize the former primary.

## Related
- [[Scaling]]
- [[Horizontal scaling]]
- [[Active-passive systems]]
- [[Active-active systems]]
- [[Data partitioning]]
