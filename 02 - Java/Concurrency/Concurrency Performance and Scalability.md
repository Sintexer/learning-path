# Concurrency Performance and Scalability

## Resource-Bound Tasks

A task might be [[IO-Bound vs CPU-Bound|I/O-bound, CPU-bound]], network-bound, or DB-bound. We say a task is bound to a resource when that resource becomes the ceiling that prevents improving throughput by adding more threads (e.g., adding more threads than available DB connections will not improve throughput and only adds queuing).

To achieve better performance we need to:
1. Utilize the processing resources we have more effectively (keep the bottleneck resource productively busy).
2. Enable our program to exploit additional processing resources if they become available.

If a program is CPU-bound, we can achieve better concurrency by adding more CPUs—but if the application doesn't keep the processors busy (due to lock contention or serialization), adding more cores won't help.

**Scalability** is the ability to improve throughput or capacity when additional computing resources (such as CPU, memory, storage, or I/O bandwidth) are added.

Poorly designed concurrent applications can behave **worse** than the same application written single-threaded (due to context-switching overhead, lock contention, and cache-line bouncing—see [[Single Writer Principle]]).

## Sizing & Contention Rules of Thumb

- **CPU-bound pools:** Size near `N_cpu` (or `N_cpu + 1`) to avoid context-switch thrashing.
- **I/O-bound pools:** Size using [[Little's Law]] (`L = λ × W`) or `N_cpu × (1 + Wait/Compute)`, bounded by downstream limits (DB connection pool size, downstream [[Rate Limiting]]). On Java 21+, [[Virtual Threads]] remove the platform-thread sizing ceiling for blocking I/O, while `Semaphore` / [[Bulkhead Pattern]] still caps downstream concurrency.
- **Amdahl's Law:** The serial (locked) fraction of a program caps maximum speedup regardless of core count. Shrink [[Critical Section|critical sections]], split locks (lock striping), replace locks with [[Compare-And-Swap|CAS]] / `LongAdder`, or eliminate shared writes via the [[Single Writer Principle]].

> [!warning] Failure modes & trade-offs
> - **Oversized thread pools hurt latency:** More threads than needed increase scheduling overhead, CPU cache eviction, and tail latency without increasing throughput—especially when blocked behind a smaller DB connection pool.
> - **Hidden serial bottlenecks:** A single `synchronized` logger, shared metrics counter, or hot DB row lock can cap an entire multi-core service via Amdahl's Law.

## Flashcards

Scalability:::The ability of a system to improve throughput or capacity when additional computing resources (CPU, memory, storage, I/O bandwidth) are added.

For ==CPU-bound== workloads, thread pool size should stay close to the number of CPU cores (`N_cpu`); adding significantly more threads degrades throughput via context switching and cache thrashing.

## Why can adding more worker threads make a DB-bound service slower instead of faster? #card
Once threads exceed the available database connections (or DB throughput), extra threads simply queue on the connection pool, increasing in-flight work (`L`) and wait time (`W`) per [[Little's Law]], while adding memory and context-switch overhead.

## Related
- [[concurrency]]
- [[IO-Bound vs CPU-Bound]]
- [[Little's Law]]
- [[Single Writer Principle]]
- [[Critical Section]]
- [[Mutual Exclusion]]
- [[Compare-And-Swap]]
- [[Virtual Threads]]
- [[Scaling]]
