---
flashcards:
  q-cxui: { nid: 1795356718292, hash: 24kzuc93, sync: ibmr7hjm }
  q-8sa9: { nid: 1795356718317, hash: nc79m9hz, sync: z28ivwge }
---

Mutual exclusion is the means by which *only one writer can have access to a protected resource at a time*, and is usually implemented with a *locking strategy*. It is the way to implement the [[Pessimistic Concurrency Control]].

Locking strategies require an *arbitrator*, usually the operating system kernel, to get involved when the contention occurs to decide who gains access and in what order.  This can be a very expensive process often requiring many more CPU cycles than the actual transaction to be applied to the business logic would use.  

Those waiting to enter the [[Critical Section]] in advance of performing the mutation must queue, and this queuing effect ([[Little's Law]]) causes latency to become unpredictable and ultimately restricts throughput.

> [!warning] Failure modes & trade-offs
> - **Lock contention & convoying:** Once threads queue on a mutex, context switches and cache invalidations dominate CPU time, collapsing throughput well below single-threaded speed.
> - **Distributed mutual exclusion is harder:** A [[Distributed Lock]] with a TTL cannot guarantee safety on its own—a GC pause or network delay can let a lock expire while the holder still thinks it owns it. Pair distributed locks with a monotonic [[Fencing Token]] validated by the storage layer.
> - **Latch vs. Lock:** Inside databases, short-lived memory-structure protection uses a [[Database Latch]], whereas transaction-duration logical isolation uses a [[Database Lock]].

## Flashcards

Mutual exclusion:::Guarantee that at most one execution context accesses a [[Critical Section]] or mutates a protected resource at a time.
^q-cxui

## Why is a TTL-based distributed lock alone insufficient to guarantee mutual exclusion across services? #card
A process holding the [[Distributed Lock]] can pause (e.g. JVM stop-the-world GC or OS scheduling delay) longer than the lease TTL. Another node acquires the lock, and when the paused node resumes it can still write stale data. Preventing corruption requires a monotonic [[Fencing Token]] checked by the resource/storage layer.
^q-8sa9

## Related
- [[Critical Section]]
- [[Pessimistic Concurrency Control]]
- [[Single Writer Principle]]
- [[Compare-And-Swap]]
- [[Little's Law]]
- [[Distributed Lock]]
- [[Fencing Token]]
- [[Database Latch]]
- [[Database Lock]]