# Concurrent Collections and Synchronizers

Notes on `java.util.concurrent` collections, blocking queues, interruption handling, and synchronizers (based on *Java Concurrency in Practice*, Ch. 5).

## Synchronized vs. Concurrent Collections

- **Synchronized wrappers** (`Vector`, `Hashtable`, `Collections.synchronized*`) lock the entire collection on every method call.
  - Compound actions (iteration, check-then-act `putIfAbsent`, `getLast` via `size() - 1`) still require **client-side locking**—otherwise concurrent mutation throws `ArrayIndexOutOfBoundsException` or `ConcurrentModificationException` (fail-fast iterators).
  - Hidden iteration happens in `hashCode()`, `equals()`, `toString()`, `containsAll()`, and copy constructors.
  - Holding a collection lock while calling external code (`doSomething(elem)`) risks deadlocks.
- **Concurrent collections** (`ConcurrentHashMap`, `CopyOnWriteArrayList`, `ConcurrentLinkedQueue`) replace coarse global locking with finer-grained concurrency and dramatically higher scalability.

### `ConcurrentHashMap`
- Uses fine-grained locking / CAS (lock striping in earlier JDKs; per-bin CAS + node locking in Java 8+): arbitrary concurrent readers, concurrent readers with writers, and multiple concurrent writers.
- **Weakly consistent iterators:** Tolerate concurrent modification without throwing `ConcurrentModificationException`.
- `size()` and `isEmpty()` are approximations (moving targets in concurrent systems), trading exactness for fast `get`/`put`/`remove`.
- Exposes atomic compound operations via `ConcurrentMap` (`putIfAbsent`, `remove(k, v)`, `replace`, `computeIfAbsent`) because you cannot lock the whole map externally.

### Copy-On-Write Collections (`CopyOnWriteArrayList`, `CopyOnWriteArraySet`)
- Every mutation copies the underlying array and safely publishes the new immutable snapshot.
- Iterators never throw `ConcurrentModificationException` and see the exact snapshot at iterator creation time.
- **Use only when reads/iterations vastly outnumber writes** (e.g. event listener registries).

## Blocking Queues & Producer-Consumer

- `BlockingQueue` coordinates producer/consumer control flow: `put`/`take` block when full/empty; `offer`/`poll` support non-blocking or timed attempts (enabling load shedding / [[Backpressure]]).
- **Bounded queues** are a vital resource-management tool—unbounded queues never apply [[Backpressure]] and risk OOM under overload.
- **`SynchronousQueue`:** Zero-capacity handoff queue—a producer blocks until a consumer directly takes the item, minimizing handoff latency and signaling that a consumer accepted responsibility.
- **Work stealing (`Deque` / `BlockingDeque`):** Each worker has its own deque; when idle, it steals from the tail of another worker's deque. Ideal when consumers also produce new tasks (crawlers, graph traversal, `ForkJoinPool`).

## Handling `InterruptedException`

When calling a blocking method that throws `InterruptedException`, you have two valid options:
1. **Propagate** `InterruptedException` to the caller (after brief cleanup if needed).
2. **Restore the interrupt status** if the method signature cannot throw it (e.g. `Runnable.run()`): catch `InterruptedException` and call `Thread.currentThread().interrupt()`.
- **Never swallow `InterruptedException`** (catch-and-ignore hides cancellation from code higher up the call stack).

## Synchronizers

- **Latch (`CountDownLatch`):** Single-use gate. Threads wait until the count reaches zero, after which the gate stays open forever (e.g. waiting for service initialization or starting workers simultaneously).
- **`FutureTask`:** Acts like a latch wrapping a `Callable` (states: waiting → running → completed). `Future.get()` blocks until completion and wraps task exceptions in `ExecutionException`.
- **Counting `Semaphore`:** Controls how many concurrent activities can hold a permit to a resource (resource pools, bounded collections, or capping downstream concurrency with [[Virtual Threads]] / [[Bulkhead Pattern]]).
- **`CyclicBarrier`:** Reusable rendezvous point for a fixed number of parties in iterative parallel algorithms (e.g. simulation steps). If any party times out or is interrupted, the barrier breaks (`BrokenBarrierException`).
- **`Exchanger`:** Two-party barrier where two threads swap objects (e.g. full buffer for empty buffer), providing safe publication for both.

> [!warning] Failure modes & trade-offs
> - **`ConcurrentHashMap.computeIfAbsent` deadlock / contention:** The mapping function executes **while holding the hash bin lock**. Never call blocking I/O or recursively update the same map inside `computeIfAbsent` (can deadlock or stall unrelated keys sharing the bin).
> - **Unbounded `LinkedBlockingQueue` OOM:** Using unbounded queues (including `Executors.newFixedThreadPool`) hides overload until heap exhausts—always bound queues and define a rejection/[[Backpressure]] policy.
> - **Copy-on-write write amplification:** Mutating a large `CopyOnWriteArrayList` frequently causes O(n) array copies and heavy GC churn.

## Flashcards

What are the two valid ways to handle `InterruptedException` in library or application code?::Either propagate `InterruptedException` to the caller, or catch it and restore the interrupt flag via `Thread.currentThread().interrupt()`—never swallow it silently.

`CountDownLatch` is a ==single-use== gate that stays open once it reaches zero, whereas `CyclicBarrier` is ==reusable== across multiple phases/generations.

## Why is calling blocking I/O inside `ConcurrentHashMap.computeIfAbsent()` dangerous? #card
`computeIfAbsent` holds the lock on the target hash bin for the entire duration of the mapping lambda. Blocking I/O stalls all other threads writing to keys in that bin, and recursive updates to the same map can cause a deadlock.

## When should you use `CopyOnWriteArrayList` vs. `ConcurrentHashMap`? #card
Use `CopyOnWriteArrayList` only when iterations vastly outnumber writes (e.g. listener lists), because every mutation copies the whole array. Use `ConcurrentHashMap` for general-purpose concurrent key-value access with frequent reads and writes.

## Related
- [[concurrency]]
- [[Safe Publication and Thread-Safe Design]]
- [[Executors and Task Execution]]
- [[Backpressure]]
- [[Bulkhead Pattern]]
- [[Compare-And-Swap]]
- [[Virtual Threads]]
