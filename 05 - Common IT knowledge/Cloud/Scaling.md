**Scaling** is the ability of a system to handle growing load (requests per second, data volume, or concurrent users) while maintaining acceptable latency and [[Availability|availability]].

Scaling can be [[Horizontal scaling|horizontal]] (scaling out/in: adding or removing instances) or [[Vertical scaling|vertical]] (scaling up/down: increasing CPU, RAM, or disk on a single node).

## Horizontal vs Vertical Scaling

| Dimension | [[Vertical scaling|Vertical (Scale Up)]] | [[Horizontal scaling|Horizontal (Scale Out)]] |
| --- | --- | --- |
| **Mechanism** | Bigger machine (more CPU/RAM/IOPS) | More machines behind [[Load Balancing]] |
| **App changes** | Minimal; works for stateful apps | Requires stateless compute or [[Data partitioning]] |
| **Ceiling** | Hard hardware & cost limit | Virtually unbounded (until shared bottlenecks hit) |
| **Fault tolerance** | Single point of failure unless paired with [[Active-passive systems]] | Natural redundancy ([[Active-active systems]]) |
| **Downtime** | Often requires restart/failover | Zero-downtime elastic scaling |

## Scaling Dimensions (AKF Scale Cube)

- **X-axis (Cloning)**: run identical stateless copies behind a load balancer ([[Horizontal scaling]]).
- **Y-axis (Functional decomposition)**: split by domain/service ([[Microservices App]]).
- **Z-axis (Data partitioning)**: shard data by customer/key ([[Data partitioning]]).

> [!tip] Production reality
> Scaling vertically first is often the pragmatic choice early on—modern single machines handle enormous throughput with zero distributed-systems complexity. Move to horizontal scaling and sharding once you hit hardware limits, availability requirements, or team-autonomy bottlenecks. Always verify scaling with [[Little's Law]] (\(L = \lambda W\)) and watch for non-scaling bottlenecks (single-leader DB writes, global locks, DNS/LB limits).

## Flashcards

What are the three axes of the AKF Scale Cube?::X-axis: horizontal cloning (stateless replicas); Y-axis: functional decomposition (microservices); Z-axis: data partitioning (sharding by key).

Vertical scaling increases ==resources on a single node== (CPU/RAM), whereas horizontal scaling adds ==more independent nodes== behind a load balancer.

## When is vertical scaling preferable to horizontal scaling? #card

When the workload is stateful or single-writer (e.g., an OLTP primary database), traffic fits comfortably within modern machine limits, and avoiding distributed coordination ([[Replication Lag]], sharding, [[Split-Brain]]) saves major engineering complexity.

## Related
- [[Horizontal scaling]]
- [[Vertical scaling]]
- [[Load Balancing]]
- [[Data partitioning]]
- [[Reliability, Scalability, Maintainability]]
- [[Little's Law]]
