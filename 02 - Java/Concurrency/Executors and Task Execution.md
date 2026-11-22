# Executors and Task Execution

Notes on decoupling task submission from execution policy in Java (based on *Java Concurrency in Practice*, Ch. 6, updated with modern Loom primitives).

## Why Not Thread-Per-Task (Platform Threads)?

- **Thread lifecycle overhead:** Creating and tearing down OS-backed platform threads is expensive in CPU and latency.
- **Resource exhaustion:** Unbounded platform threads consume stack memory (`-Xss` per thread; see [[jvmArgs]]), saturate CPU via context switching, and degrade stability under load spikes.

## The `Executor` Framework

Decouples *what* work runs from *how* (thread pool sizing, queueing, rejection, shutdown).

Common `Executors` factory methods (and their caveats):
- `newFixedThreadPool(n)`: Fixed `n` threads backed by an **unbounded** `LinkedBlockingQueue`.
- `newCachedThreadPool()`: Reaps idle threads and creates new ones on demand with **no upper bound** on thread count (uses `SynchronousQueue`).
- `newSingleThreadExecutor()`: One worker thread processing tasks sequentially in queue order (replaces the worker if it dies).
- `newScheduledThreadPool(n)`: Delayed and periodic task execution.
- `newVirtualThreadPerTaskExecutor()` (Java 21+, [[Virtual Threads]]): Spawns a lightweight JVM-managed virtual thread per task without pooling.

### Graceful vs. Abrupt Shutdown
- `shutdown()` stops accepting new tasks and finishes queued/in-flight work (graceful).
- `shutdownNow()` attempts to cancel running tasks via interruption and returns unstarted queued tasks (abrupt).

## `Timer` vs. `ScheduledThreadPoolExecutor`

Avoid `java.util.Timer` in favor of `ScheduledThreadPoolExecutor`:
1. `Timer` uses a **single thread**—one slow task delays all other scheduled tasks.
2. If any `TimerTask` throws an unchecked exception, the `Timer` thread dies permanently and cancels the entire `Timer`.
3. For custom scheduling, `DelayQueue` provides delay-ordered blocking retrieval.

## Result-Bearing Tasks: `Callable`, `Future`, `CompletionService`

- **`Callable<V>`** returns a value and may throw checked exceptions (unlike `Runnable`).
- **`Future<V>`** tracks task lifecycle (`isDone`, `isCancelled`, `cancel`, `get`). `get()` blocks until completion and wraps failures in `ExecutionException`.
- **`ExecutorCompletionService`:** Combines an `Executor` with a `BlockingQueue` of completed `Future`s so callers can process results **as they finish** (`take()`/`poll()`) rather than blocking on the slowest task first.
- **Time budgets:** Use timed `Future.get(timeout, unit)` or `ExecutorService.invokeAll(tasks, timeout, unit)` (which cancels unfinished tasks on timeout) to enforce end-to-end [[Timeout Pattern|timeouts]].

> [!warning] Failure modes & trade-offs
> - **Avoid `Executors.newFixedThreadPool` and `newCachedThreadPool` in production:** `newFixedThreadPool` uses an unbounded queue (OOM risk under overload); `newCachedThreadPool` has unbounded threads (CPU/stack exhaustion under bursty load). Construct `ThreadPoolExecutor` explicitly with a **bounded queue**, named `ThreadFactory`, and an explicit `RejectedExecutionHandler` (e.g. `AbortPolicy` or `CallerRunsPolicy` for [[Backpressure]]).
> - **Thread starvation deadlock:** Submitting tasks that wait on results of subtasks submitted to the **same** fixed thread pool can deadlock when all threads are blocked waiting for queued subtasks.
> - **Modern alternative (Java 21–25):** For [[IO-Bound vs CPU-Bound|I/O-bound]] fan-out/fan-in, prefer [[Virtual Threads]] + `StructuredTaskScope` (final in Java 25; see [[Java Evolution]]) over raw `ExecutorService` or `CompletableFuture.allOf()`, because sibling tasks are automatically cancelled on failure.

## Flashcards

## Why are `Executors.newFixedThreadPool()` and `Executors.newCachedThreadPool()` risky in production? #card
`newFixedThreadPool` uses an **unbounded** `LinkedBlockingQueue` (risking OOM when arrival rate exceeds service rate), while `newCachedThreadPool` allows **unbounded** platform threads (risking thread/memory exhaustion under traffic spikes). Use an explicit `ThreadPoolExecutor` with bounded queue + rejection policy.

Why should you use `ScheduledThreadPoolExecutor` instead of `java.util.Timer`?::`Timer` runs all tasks on a single thread (so one slow task delays all others) and permanently dies if any task throws an unchecked exception.

What advantage does `ExecutorCompletionService` have over iterating a `List<Future<V>>` and calling `get()`?::It places completed `Future`s onto an internal queue as they finish, letting callers process fastest results immediately instead of blocking behind the slowest task in submission order.

Never submit dependent subtasks to the same fixed-size thread pool that the parent task is blocking on—it causes thread-starvation deadlock. #card-reminder

## Related
- [[concurrency]]
- [[Concurrent Collections and Synchronizers]]
- [[Concurrency Performance and Scalability]]
- [[Java Evolution]]
- [[Virtual Threads]]
- [[Timeout Pattern]]
- [[Backpressure]]
- [[Bulkhead Pattern]]
- [[Little's Law]]
