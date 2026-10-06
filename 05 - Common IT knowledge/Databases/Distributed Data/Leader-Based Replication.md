This mode of [[Replication]] is usually a built-in feature of many relational databases. Under the hood, in leader-based replication leader feeds the [[Replication Log]] to the followers. 
## Leader

One of the replicas is designated the *leader*. When clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage.

Only The leader accepts writes in this solution.

## Follower

Non-leader replicas receive the data from the leader whenever it writes new data to its local storage. This event stream is called *replication log* or *change stream*. Followers apply writes in the same order as they were processed on the leader.

When a client wants to read from the database, it can query either the leader or any of the followers. However, writes are only accepted on the leader.

To spawn new followers you need the way to quickly capture the database snapshot, most databases have this feature natively (required for backups). New follower takes this snapshot and requests all the writes since from the leader. After processing the backlog of the data since last snapshot, follower starts processing live data.