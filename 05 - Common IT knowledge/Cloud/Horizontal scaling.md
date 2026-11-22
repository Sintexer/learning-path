**Add additional instances (scale out).** For the application to work in an [[Active-active systems|active-active system]], it is created as stateless, not storing any client sessions on the server. This means that having two or four servers wouldn't require any application changes. It would only be a matter of creating more instances when required and shutting them down when traffic decreases.

There are many more advantages to using an active-active system in comparison with an [[Active-passive systems|active-passive system]]. Modifying your application to become stateless provides scalability and eliminates single-node hardware ceilings.

## What Horizontal Scaling Requires

- **[[Load Balancing]]**: distributes traffic across instances and removes unhealthy ones via [[Health Checks]].
- **Stateless compute**: sessions and shared caches live in external stores (e.g., [[Redis]]) rather than local RAM.
- **Data-tier scaling**: stateless app servers scale easily, but the underlying [[Database]] often becomes the bottleneck and eventually requires read replicas ([[Replication]]) or [[Data partitioning]] (sharding).

> [!warning] Failure modes & trade-offs
> - **Downstream amplification**: horizontally scaling stateless pods from 10 to 100 multiplies DB connection pools by 10x and can exhaust [[Database]] connections unless you use connection pooling (e.g., PgBouncer) and [[Backpressure]].
> - **Autoscaling lag**: if traffic spikes faster than container boot + warmup time (especially for JVM apps), scale-out arrives too late—pair reactive autoscaling with headroom or [[Rate Limiting]].
> - **Coordination overhead**: distributed jobs need [[Distributed Lock|distributed locks]] or leader election so multiple replicas don't duplicate work.

## Flashcards

Horizontal scaling (scaling out):::Adding more independent instances behind a [[Load Balancing|load balancer]] to handle higher throughput and improve fault tolerance.

Scaling stateless app servers horizontally frequently shifts the bottleneck to the ==[[Database]]== (connection pool exhaustion or write throughput limits).

Before scaling stateless services out, always check downstream connection pool limits and database write capacity. #card-reminder

### Why can reactive horizontal autoscaling fail during sudden traffic spikes? #card

Because detecting high load, provisioning nodes/pods, pulling images, and warming up the runtime (e.g., JVM JIT) takes seconds to minutes; by the time new replicas are ready, existing replicas may already be overwhelmed.

## Related
- [[Scaling]]
- [[Vertical scaling]]
- [[Active-active systems]]
- [[Load Balancing]]
- [[Data partitioning]]
- [[Backpressure]]
