With an **active-passive system**, only one of the two instances is [[Availability|available]] to serve live traffic at a time while the standby instance waits to take over on failure. One advantage of this method is that for stateful applications (where data about the client's session is stored on the server), there won't be any issues. This is because the customers are always sent to the server where their session is stored. Suitable for [[Vertical scaling]] and single-writer databases using [[Leader-Based Replication]].

Standby modes differ by recovery speed (RTO) and cost:
- **Cold standby**: infrastructure is provisioned or booted only after failure (cheapest, slowest failover).
- **Warm standby**: standby is running at reduced capacity and receives periodic backups or async [[Replication]] updates.
- **Hot standby**: standby is fully provisioned and continuously synced via [[Synchronous vs Asynchronous Replication|synchronous or near-real-time replication]] for fast failover.

> [!warning] Failure modes & trade-offs
> - **Untested failover**: passive nodes drift in configuration or stay cold (empty caches, uninitialized connection pools), causing the standby to collapse under sudden full traffic.
> - **[[Split-Brain]] during failover**: if the old active node is merely partitioned (not dead) and keeps accepting writes while the passive promotes itself, data diverges unless prevented by [[Fencing Token|fencing tokens]] or [[Quorum Consensus]].
> - **Data loss (RPO > 0)**: async [[Replication Lag]] means unshipped transactions on the failed primary are lost upon promotion.

## Flashcards

Active-passive architecture::A high-availability topology where one primary node handles all live traffic while a standby node takes over only after primary failure.

The biggest operational risk in active-passive systems is ==[[Split-Brain]]== (both nodes acting as primary) or ==untested failover== where the cold standby collapses under full traffic.

Practice failover drills regularly; an untested active-passive standby is not real redundancy. #card-reminder

### What is the difference between cold, warm, and hot standby in active-passive systems? #card

- **Cold**: provisioned on demand after failure (lowest cost, highest RTO).
- **Warm**: running at reduced scale with async replication (moderate cost and RTO).
- **Hot**: full capacity synced in near real time (highest cost, lowest RTO/RPO).

## Related
- [[Active-active systems]]
- [[Availability]]
- [[Vertical scaling]]
- [[Leader-Based Replication]]
- [[Fencing Token]]
- [[Split-Brain]]
