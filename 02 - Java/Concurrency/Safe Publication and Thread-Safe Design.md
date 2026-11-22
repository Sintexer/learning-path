# Safe Publication and Thread-Safe Design

Notes on safely sharing objects across threads and composing thread-safe classes (based on *Java Concurrency in Practice*, Ch. 3–4, and the [[Java Memory Model]]).

## Publication and Escape

- **Publication** makes an object accessible to code outside its current scope: storing a reference in a `public static` field, returning it from a non-private method, storing it in a non-private field of a published object, passing it to an **alien method** (any method whose behavior is not fully specified by the class, including overridable methods), or publishing an inner class instance (which carries an implicit reference to the enclosing `this`).
- **Thread confinement:** If data is accessed from only a single thread, no synchronization is needed (e.g. local variables, `ThreadLocal`, or [[Single Writer Principle]] designs).

## Immutability & Safe Publication

An object is **immutable** if:
1. Its state cannot be modified after construction.
2. All its fields are `final`.
3. It is properly constructed (the `this` reference does not escape during construction).

Even though constructor writes appear to be the "first" writes, `Object`'s constructor initializes all fields to default values first—without safe publication, another thread can observe default/stale values.

A properly constructed object can be **safely published** by:
- Initializing an object reference from a `static` initializer (guaranteed by JVM class-initialization synchronization).
- Storing a reference into a `volatile` field or `AtomicReference`.
- Storing a reference into a `final` field of a properly constructed object.
- Storing a reference into a field properly guarded by a lock.
- Placing it into a thread-safe collection (`ConcurrentHashMap`, `Collections.synchronized*`, `CopyOnWriteArrayList`, `BlockingQueue`, `ConcurrentLinkedQueue`).

**Publication rules by mutability:**
- **Immutable objects:** can be published through any mechanism.
- **Effectively immutable objects** (not `final`, but never mutated after publication): must be **safely published**, then need no further synchronization.
- **Mutable objects:** must be **safely published** AND either be thread-safe or guarded by a lock.

## Sharing Policies

1. **Thread-confined:** Owned exclusively by one thread.
2. **Shared read-only:** Immutable or effectively immutable; freely read without locks.
3. **Shared thread-safe:** Synchronizes internally; clients call public API directly.
4. **Guarded:** Accessible only while holding a specific documented lock.

## Designing Thread-Safe Classes

Three steps:
1. Identify the variables that form the object's state.
2. Identify the **invariants** constraining state variables and **postconditions** constraining valid transitions (e.g. `count` must never be negative; `17 → 18` is a compound read-modify-write action).
3. Establish and document a synchronization policy for concurrent access.

### Instance Confinement & Private Locks
- Encapsulating mutable state inside a class confines access to its methods, making thread safety verifiable locally.
- Prefer a **private lock object** (`private final Object lock = new Object();`) over synchronizing on `this`. A public intrinsic lock lets external callers acquire it and risk deadlocks or liveness bugs.

### Delegating Thread Safety
- A class can delegate thread safety to a single thread-safe variable (e.g. `AtomicLong`), or to **multiple independent** thread-safe variables **only if** no invariant couples them.
- If an invariant spans multiple fields (e.g. `lower <= upper` in `NumberRange`), independent atomic variables are **not enough**—each check-then-act (`[[Check-Then-Act]]`) must be guarded atomically under one lock.

### Extending Thread-Safe Classes
1. **Modify the original class** (safest—keeps the synchronization policy in one file).
2. **Subclass** (fragile—splits the synchronization policy across files; breaks if the superclass changes its lock).
3. **Client-side locking** (helper class locking on the **exact same lock** the underlying class uses, e.g. `synchronized (list)` for `Collections.synchronizedList`).
4. **Composition** (wrap the object and guard all access with the wrapper's own lock—most robust; see [[Composition Over Inheritance]]).

> [!warning] Failure modes & trade-offs
> - **Escaping `this` in a constructor:** Registering an event listener or starting a thread inside a constructor breaks `final`-field safe publication guarantees in the [[Java Memory Model]]. Use a private constructor + static factory method instead.
> - **Split ownership in containers:** Storing mutable objects in a thread-safe map (`ConcurrentHashMap` / `ServletContext`) only protects the map structure, not the stored objects themselves.
> - **Wrong lock in client-side locking:** Locking on a helper object instead of the wrapped collection's own monitor gives an illusion of thread safety while leaving races wide open.

## Flashcards

What are the three requirements for an object to be truly immutable in Java?::Its state cannot be modified after construction, all its fields are `final`, and `this` does not escape during construction.

## What are the four primary ways to safely publish a properly constructed object in Java? #card
1. Initialize the reference from a `static` initializer.
2. Store it into a `volatile` field or `AtomicReference`.
3. Store it into a `final` field of a properly constructed object.
4. Store it into a field properly guarded by a lock (or insert into a thread-safe `java.util.concurrent` collection).

## When can a class safely delegate thread safety to multiple underlying thread-safe fields? #card
Only when those state variables are **independent**—no class invariant constrains their values together and no operation has compound state transitions across them. If an invariant couples them (like `lower <= upper`), you must guard compound operations under a single lock.

Prefer a private lock object over `synchronized (this)` so external callers cannot acquire your internal monitor and cause deadlocks. #card-reminder

## Related
- [[concurrency]]
- [[Java Memory Model]]
- [[Check-Then-Act]]
- [[Critical Section]]
- [[Mutual Exclusion]]
- [[Single Writer Principle]]
- [[Composition Over Inheritance]]
