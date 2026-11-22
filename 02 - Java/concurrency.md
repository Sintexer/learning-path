# Java Concurrency Hub

Core Java concurrency notes, split into focused topics (primarily synthesized from *Java Concurrency in Practice* and modern Java 21–25 Loom updates).

## Topics

- [[Safe Publication and Thread-Safe Design]] — publication, escape, immutability, safe publication rules, confinement, delegating thread safety, and client-side locking.
- [[Concurrent Collections and Synchronizers]] — `ConcurrentHashMap`, `CopyOnWrite*`, `BlockingQueue`, `SynchronousQueue`, work stealing, `InterruptedException`, latches, semaphores, and barriers.
- [[Executors and Task Execution]] — `Executor` framework, thread pool pitfalls, `ScheduledThreadPoolExecutor`, `Future`, `CompletionService`, time budgets, and Loom (`[[Virtual Threads]]` + `StructuredTaskScope`).
- [[Concurrency Performance and Scalability]] — [[IO-Bound vs CPU-Bound]], scalability, Amdahl's Law, and pool sizing with [[Little's Law]].

## Related
- [[Java Memory Model]]
- [[Java Evolution]]
- [[Virtual Threads]]
- [[Compare-And-Swap]]
- [[Critical Section]]
- [[Mutual Exclusion]]
- [[Single Writer Principle]]
- [[Little's Law]]
- [[Backpressure]]
