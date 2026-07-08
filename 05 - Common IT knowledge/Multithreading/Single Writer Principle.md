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