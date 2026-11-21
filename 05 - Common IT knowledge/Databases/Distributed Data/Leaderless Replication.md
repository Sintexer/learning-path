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