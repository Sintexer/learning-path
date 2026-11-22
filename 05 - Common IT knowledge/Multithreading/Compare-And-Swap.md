---
aliases:
  - CAS
flashcards:
  q-xdci: { nid: 1795356718418, hash: 8tym7gm5, sync: vfe5pdqw }
  q-sesc: { nid: 1795356718443, hash: ttv5d9cz, sync: i8kkm5y2 }
  q-hcs6: { nid: 1795356718467, hash: ajvzypii, sync: 5z2dn53i }
---
**CAS** stands for **Compare-And-Swap**. It is a way to update a piece of data without using "locks" (which can be slow and cause programs to freeze). It is a concrete technology that makes [[Optimistic Concurrency Control]] possible.

> [!tip]
> **CAS is not a system call, and it is not implemented by the OS.** It is implemented by the **CPU hardware itself.**

Think of it like a **"conditional update"** based on a promise.

### How it works (The 3-Step Process)

Imagine you want to update a value (e.g., a bank balance) from **100 to 150**.

1.  **Read:** You look at the current value. You see it is **100**.
2.  **Compare:** You ask the system: *"Is the value still 100?"*
3.  **Swap:** 
	*   **If YES:** You change it to **150**. (Success!)
	*   **If NO:** Someone else changed it while you were looking. You don't update it; instead, you start over and try again.

### Why use it?

*   **No Locking:** You don't have to "lock" the data and make other threads wait. You just try to update it, and if someone else got there first, you just try again.
*   **Efficiency:** It is much faster than traditional locking because it avoids the "stop-and-wait" overhead.

**In one sentence:** CAS is a way of saying, *"Update this value only if it hasn't changed since I last looked at it."*

> [!warning] Failure modes & trade-offs
> - **ABA problem:** A thread reads `A`, gets preempted while another thread changes `A → B → A`, and the original CAS still succeeds even though intermediate state mutated. Prevent this with a monotonic version/stamp (e.g. Java's `AtomicStampedReference` or a [[Version Guard]]).
> - **High-contention spin / retry storms:** Under heavy multi-core contention, failed CAS loops burn CPU cycles and cause cache-line bouncing across cores. For hot counters, prefer striped cells (`LongAdder`) or redesign around the [[Single Writer Principle]].
> - **Single-word scope:** Hardware CAS atomically updates one memory word (or double-word on some ISAs); multi-field invariants still need immutable snapshots swapped by reference, locks, or [[Database Lock|database transactions]].

## Flashcards

Compare-And-Swap (CAS) is implemented by ==CPU hardware instructions== (not the OS kernel), enabling lock-free [[Optimistic Concurrency Control]].
^q-xdci

## What is the ABA problem in Compare-And-Swap, and how is it mitigated? #card
A thread reads value `A`, is paused while other threads mutate `A → B → A`, and then its CAS succeeds because the current value still matches `A` even though the underlying state changed (dangerous in lock-free linked structures with node reuse). Mitigate by pairing the value with a monotonically increasing stamp/version (e.g. `AtomicStampedReference`).
^q-sesc

Under high contention, prefer striped accumulators (like `LongAdder`) or the [[Single Writer Principle]] over hot CAS retry loops that thrash CPU caches. #card-reminder
^q-hcs6

## Related
- [[Optimistic Concurrency Control]]
- [[Atomic Conditional Mutation]]
- [[Single Writer Principle]]
- [[Mutual Exclusion]]
- [[Java Memory Model]]
- [[concurrency]]