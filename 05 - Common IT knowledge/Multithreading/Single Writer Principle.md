---
flashcards:
  q-drcv: { nid: 1795356718216, hash: h5tmy6ha, sync: w8hbxfvk }
  q-7dib: { nid: 1795356718242, hash: 5ymw9rpj, sync: 2a8aab63 }
  q-qcs8: { nid: 1795356718267, hash: 6mbt5px9, sync: 24sghihb }
---

Modern CPUs are fast, but they are limited by memory speed. When a CPU needs data that isn't in its cache, it must fetch it from main memory, causing a "stall" that can last hundreds of cycles. This is the single biggest bottleneck in CPU performance today.

This directly impacts concurrency. When multiple threads use **locks** to protect shared data, they rely on [[Compare-And-Swap|CAS]] instructions to coordinate access. Even without conflicts, these instructions are expensive - they can take tens of cycles and force the CPU to pause its optimized execution pipeline.

The situation gets significantly worse when threads _do_ conflict:

1. The OS Kernel intervenes to put the "losing" thread to sleep.
2. This causes **cache pollution**—the thread's "warm" data is evicted from the cache while it waits.
3. When the thread wakes up (often on a different core), it faces a "cold" cache and must reload all its data from memory, causing more stalls.

Benchmarks show this effect clearly: incrementing a shared counter with two threads using locks is not just twice as slow as one thread—it can be **hundreds of times slower**. This is because the system spends far more time managing _who gets access_ than actually processing data.

While this specific "counter" test seems trivial, it exposes a scaling limit. In real applications with much larger state, this same contention effect causes massive performance drops, as the "warm" cache data is constantly evicted and reloaded during context switches.

**The Bottom Line:** As we rely more on multi-core processors, traditional locking mechanisms become a liability. The overhead of coordinating threads can easily outweigh the benefit of adding more cores, meaning a new approach to concurrency is required to scale effectively.

## Single Writer Designs  
  
**Single Writer Principle:** Any given piece of data or resource should be mutated by only one execution context, ever.

If you design systems around independent components that own their own state and communicate only via messages (not shared writes), contention disappears naturally. If migrating fully isn't possible, the practical fix when facing scalability issues is to ask: _"How can I restructure this so only one thread/owner ever writes this data?"_

> [!tip] Production reality
> - **Where it shines in practice:** LMAX Disruptor ring buffers, Actor frameworks, Redis's single-threaded command loop, and per-key ordering in a [[Kafka Partition]] all leverage single-writer ownership to avoid lock contention.
> - **Watch out for false sharing:** Even if Thread A only writes field `x` and Thread B only writes field `y`, if `x` and `y` share the same 64-byte CPU cache line, the cores still invalidate each other's cache line on every write. Mitigate by padding hot independent fields (e.g. `@Contended` in the JVM).
> - **Visibility still matters:** A single writer eliminates write-write races, not read-write visibility bugs—readers still need proper [[Java Memory Model|happens-before]] publication (e.g. `volatile` / acquire-release fences).

## Flashcards

Single Writer Principle:::Design rule stating that any given piece of data or resource should be mutated by only one execution context, eliminating write-write lock contention.
^q-drcv

## Why can a lock-protected shared counter be orders of magnitude slower with two threads than with one? #card
Contended locks trigger OS kernel arbitration, thread context switches, and CPU cache invalidation/pollution. Cores spend far more cycles bouncing cache lines and reloading evicted "warm" state from main memory than executing the actual increment.
^q-7dib

## What is false sharing, and how can it hurt even a Single Writer design? #card
False sharing happens when two threads independently mutate different variables that sit on the same CPU cache line (typically 64 bytes). The CPU cache-coherency protocol invalidates the whole line across cores on every write, recreating contention; fix by padding independent hot fields onto separate cache lines.
^q-qcs8

## Related
- [[Compare-And-Swap]]
- [[Mutual Exclusion]]
- [[Critical Section]]
- [[Java Memory Model]]
- [[Kafka Partition]]
- [[Event Sourcing]]
- [[concurrency]]