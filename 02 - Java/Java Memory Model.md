The **Java Memory Model (JMM)** is the set of rules that defines what one thread is allowed to observe when multiple threads access shared memory.

It answers questions like:

- When must a write by one thread become observable by another?
- Can operations be reordered?
- Which operations are atomic?
- What does `volatile` guarantee?
- What does locking guarantee?
- What happens when code has a data race?

The JMM is an **abstract behavioral model**. It does not say “flush this cache line” or “execute this CPU instruction.” It defines which outcomes are legal. The JVM, JIT compiler, and CPU must implement behavior consistent with those rules.

## Happens-before rule

Suppose action `A` happens-before action `B`:
	
```
A happens-before B
```

This means:

1. `A` is ordered before `B` according to the JMM.
2. The effects of `A` must be visible to `B`.
3. The compiler and CPU may not reorder operations in a way that violates that relationship.

A useful informal interpretation is:

> If A happens-before B, B is not allowed to behave as though A’s relevant effects never happened.

### Core Happens-Before Rules

1. **Program order:** Within a single thread, each action happens-before every subsequent action in program order.
2. **Monitor lock:** An unlock on a monitor happens-before every subsequent lock on that same monitor (`synchronized`, `ReentrantLock`).
3. **`volatile` variable:** A write to a `volatile` field happens-before every subsequent read of that same field (including all writes made by the writing thread *before* the `volatile` write).
4. **Thread lifecycle:** `Thread.start()` happens-before any action in the started thread; all actions in a thread happen-before another thread successfully returns from `Thread.join()` on it.
5. **Transitivity:** If `A` happens-before `B` and `B` happens-before `C`, then `A` happens-before `C`.
6. **Safe publication of `final` fields:** Properly constructed objects (where `this` does not escape the constructor) guarantee other threads see initialized `final` fields without extra synchronization.

> [!warning] Failure modes & trade-offs
> - **`volatile` does not make compound actions atomic:** `volatile int count; count++;` is still a read-modify-write race (use `AtomicInteger` / [[Compare-And-Swap|CAS]] or `LongAdder`).
> - **Broken Double-Checked Locking (DCL):** Without `volatile` on the singleton field, JIT/CPU instruction reordering can publish a partially constructed object reference to another thread.
> - **False sharing:** Even with correct JMM visibility, two threads mutating independent `volatile` fields on the same 64-byte cache line thrash CPU caches—pad hot fields (`@Contended`) or redesign around the [[Single Writer Principle]].
> - **Escaping `this` in a constructor:** Registering listeners or starting threads inside a constructor breaks `final`-field safe publication guarantees.

## Flashcards

What does `A happens-before B` guarantee in the Java Memory Model?::The effects of action `A` are guaranteed to be ==visible== to `B`, and neither the compiler nor CPU may reorder operations in a way that violates that ordering.

## Why does Double-Checked Locking in Java require the instance field to be `volatile`? #card
Without `volatile`, object allocation and constructor execution can be reordered so another thread observes a non-null reference to a **partially constructed** object. Marking the field `volatile` establishes a happens-before edge between constructor writes (before the volatile write) and subsequent reads.

`volatile` guarantees visibility and ordering (happens-before), NOT atomicity of compound read-modify-write operations like `count++`—use `Atomic*` ([[Compare-And-Swap|CAS]]) or locks for those. #card-reminder

## Related
- [[concurrency]]
- [[Compare-And-Swap]]
- [[Single Writer Principle]]
- [[Mutual Exclusion]]
- [[Critical Section]]