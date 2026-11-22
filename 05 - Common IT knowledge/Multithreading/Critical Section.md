A **critical section** is a part of a program that accesses a shared resource (like a variable, file, or [[Database|database]]) that must not be accessed by more than one process or thread at the same time.

In short: **It is the "exclusive zone" where only one worker is allowed at a time to prevent errors.**

### Requirements for a Correct Solution

Any mechanism guarding a critical section must satisfy three properties:
1. **[[Mutual Exclusion]]:** At most one thread executes inside the critical section at a time.
2. **Progress:** If no thread is in the critical section and one or more threads want to enter, the choice of who enters next cannot be postponed indefinitely.
3. **Bounded Waiting:** A thread waiting to enter must eventually get a turn (no starvation).

### Common Ways to Protect or Eliminate It

- **Pessimistic locking ([[Pessimistic Concurrency Control]]):** Mutexes, `synchronized` blocks, `ReentrantLock`, or [[Database Lock|database locks]]. Simple to reason about, but threads block and queue when contended.
- **Lock-free atomics ([[Compare-And-Swap|CAS]]):** Replace a short read-modify-write critical section with an atomic hardware instruction (e.g. `AtomicInteger`).
- **Eliminate shared mutable state:** Partition ownership via the [[Single Writer Principle]] or immutable data structures so no critical section exists in the first place.

> [!warning] Failure modes & trade-offs
> - **Holding locks across I/O:** Calling a database, HTTP endpoint, or blocking queue inside a critical section turns microsecond lock waits into millisecond/second stalls, triggering lock convoying and thread-pool exhaustion.
> - **Lock-ordering deadlocks:** Acquiring multiple critical sections in inconsistent order across threads causes deadlock. Always enforce a global lock acquisition order or use timed `tryLock`.
> - **Priority inversion:** A low-priority thread holding a critical section gets preempted while a high-priority thread waits on it.

## Flashcards

What three properties must a correct critical section synchronization mechanism satisfy?::[[Mutual Exclusion]], Progress (no indefinite postponement when free), and Bounded Waiting (no starvation).

Never perform blocking I/O or external network calls inside a critical section—shrink the locked region to pure in-memory state transitions or eliminate it with the [[Single Writer Principle]]. #card-reminder

## Related
- [[Mutual Exclusion]]
- [[Pessimistic Concurrency Control]]
- [[Compare-And-Swap]]
- [[Single Writer Principle]]
- [[Database Latch]]
- [[concurrency]]