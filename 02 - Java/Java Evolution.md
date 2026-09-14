## Java 11 → 25: The Senior Overview

I'll group by theme rather than strict version order, since that's how you'd actually reason about adopting them. LTS releases are **11, 17, 21, 25** — most shops jump between those.

---

## 1. Language Features — Data Modeling & Pattern Matching

This is the biggest shift in "how you write Java" over this span.

**Records (Java 16, preview in 14)**

```java
record Point(int x, int y) {}
```

Immutable data carriers — auto-generates constructor, accessors, `equals`/`hashCode`/`toString`. Eliminates a massive amount of Lombok/boilerplate DTO code. Nuance: accessors are `x()`, not `getX()` — this trips people integrating with libraries expecting bean conventions (Jackson handles it fine since 2.12+, but older reflection-based tools may not).

**Sealed Classes/Interfaces (Java 17)**

```java
sealed interface Shape permits Circle, Square, Triangle {}
```

Restricts which classes can implement/extend a type. Combined with pattern matching, this gives you **exhaustiveness checking** in `switch` — the compiler knows all subtypes, so a `switch` over a sealed hierarchy with no `default` is legal and safe if all cases are covered. This is Java's answer to algebraic data types / sum types.

**Pattern Matching for `instanceof` (Java 16)**

```java
if (obj instanceof String s && s.length() > 5) { ... }
```

No more `(String) obj` casts.

**Pattern Matching for `switch` (Java 21)**

```java
String describe(Object o) {
    return switch (o) {
        case Integer i when i > 0 -> "positive int";
        case Integer i -> "non-positive int";
        case String s -> "string: " + s;
        case null -> "null!";
        default -> "unknown";
    };
}
```

`switch` can now match on type, use guards (`when`), and even match `null` explicitly (previously `switch` NPE'd on null).

**Record Patterns (Java 21)**

```java
if (obj instanceof Point(int x, int y)) {
    // x, y destructured directly
}
```

Deconstruction — genuinely useful for nested record hierarchies (e.g., matching `Point(var x, var y)` inside a `Line(Point a, Point b)`).

**Switch Expressions (Java 14) & Text Blocks (Java 15)**

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    default -> 7;
};

String json = """
    {
      "name": "Alice"
    }
    """;
```

Both are now just idiomatic Java — no reason not to use them.

**`var` (Java 10, so technically pre-11, but worth a mention since 11 was the first LTS to have it)** — local variable type inference. Senior take: use it when the right-hand side makes the type obvious (`var list = new ArrayList<String>()`), avoid it when it obscures the type (`var result = process(x)`).

**Unnamed variables & patterns (Java 21/22)**

```java
if (obj instanceof Point(var x, _)) { ... }  // ignore the y component
```

---

## 2. Concurrency — Project Loom (the headline story)

**Virtual Threads (Java 21, final — JEP 444)**  
Lightweight, JVM-managed threads (not OS threads) — you can spawn millions of them.

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> handleRequest());
}
```

This is the biggest change to Java's concurrency story since `java.util.concurrent` itself. It makes the **thread-per-request** model viable again at scale — you get the simplicity of blocking, synchronous code without the throughput ceiling of platform threads, competing directly with reactive/async frameworks (WebFlux, etc.) for I/O-bound workloads without the cognitive overhead of reactive chains.

**Nuance you already know:** `synchronized` pinning was a real problem in 21–23; **fixed in Java 24 (JEP 491)** — `synchronized` no longer pins carriers in the common case. On Java 25, this concern is largely gone.

**Structured Concurrency (Java 25, final — JEP 505, after several preview rounds)**

```java
try (var scope = StructuredTaskScope.open()) {
    var user = scope.fork(() -> fetchUser(id));
    var orders = scope.fork(() -> fetchOrders(id));
    scope.join();
    return new Response(user.get(), orders.get());
}
```

Treats a group of related virtual-thread tasks as a **single unit of work** — if one fails or the scope is cancelled, siblings are cancelled too, and errors propagate cleanly. Solves the "fire off parallel tasks, one fails, others leak" problem that plagued raw `ExecutorService` usage. This is now stable in 25 — worth building new fan-out/fan-in logic on top of it instead of `CompletableFuture.allOf()`.

**Scoped Values (Java 25, final — JEP 506)**

```java
static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

ScopedValue.where(CURRENT_USER, user).run(() -> handleRequest());
```

An immutable, structured alternative to `ThreadLocal` designed for virtual threads. `ThreadLocal` with millions of virtual threads is a memory/leak risk (each thread carries its own copy, easy to forget to clean up); `ScopedValue` is bound for the duration of a call and automatically unbound — cheaper and safer for Loom-heavy code. Senior takeaway: **default to `ScopedValue` over `ThreadLocal`** for any new context-propagation code (request-scoped user/tenant/trace-id) once you're on 25.

---

## 3. Garbage Collection & Runtime Performance

- **ZGC (Java 15, production-ready)** — sub-millisecond pause times, scales to multi-terabyte heaps, pause time independent of heap size.
- **Generational ZGC (Java 21, default in 23)** — splits young/old generations like G1, dramatically improving throughput on top of ZGC's low pause times. If you're not on Generational ZGC by 23+, you're leaving performance on the table for latency-sensitive services.
- **Shenandoah GC** — similar low-pause goals, alternative to ZGC (Red Hat–driven), matured throughout this period.
- **CDS (Class Data Sharing) / AppCDS improvements**, including **dynamic CDS archives (Java 13)** and ongoing startup-time work — relevant for containerized/serverless Java where cold-start time matters.
- **Compact Object Headers (Java 24, JEP 450, experimental)** — reduces per-object header overhead, meaningful memory savings for object-heavy workloads.

---

## 4. Native Interop & Low-Level APIs (Project Panama)

**Foreign Function & Memory API (Java 22, final — JEP 454)**  
Replaces JNI for calling native code and JNA-style off-heap memory access:

```java
try (Arena arena = Arena.ofConfined()) {
    MemorySegment segment = arena.allocate(100);
    // interact with native memory safely, no JNI boilerplate
}
```

Big deal for anyone doing native library integration, high-performance I/O, or building bindings — significantly safer and more ergonomic than JNI, with an `Arena`-based lifetime model preventing use-after-free bugs.

**Vector API (still incubating/preview through 25, JEP 508 in 25)**  
Explicit SIMD (Single Instruction Multiple Data) programming model — lets Java code compile down to vectorized CPU instructions, relevant for numerics-heavy code (ML preprocessing, data processing pipelines). Still not finalized as of 25 — worth watching, not yet safe to build production APIs around without expecting further changes.

---

## 5. Collections & Core Library

**Sequenced Collections (Java 21, JEP 431)**

```java
List<Integer> list = ...;
list.getFirst();
list.getLast();
list.reversed();
```

Finally a unified way to get first/last elements and a reversed view across `List`, `Deque`, `LinkedHashSet`, `LinkedHashMap` — previously inconsistent (`List` had no `getFirst()`, you'd reach for `get(0)`).

**`Collectors.teeing()` (Java 12)** — combine two downstream collectors into one pass, e.g. computing average via count+sum simultaneously.

**Helpful NullPointerExceptions (Java 14, JEP 358)** — NPE messages now tell you _which_ variable was null in a chained call (`a.getB().getC()` — the message says exactly which link was null). Enabled by default since Java 15. Genuinely saves debugging time; make sure it's not disabled in your prod JVM flags (`-XX:-ShowCodeDetailsInExceptionMessages` would turn it off).

---

## 6. Platform, Tooling, and Deprecations

- **Single-file source-code launching (Java 11, JEP 330)** — `java MyScript.java` runs directly without a separate compile step. Extended in Java 22/23 (JEP 458/463) to support **unnamed classes and instance main methods**, aimed at simplifying the learning curve and quick scripting:
    
```java
void main() {    System.out.println("Hello");}
```
    
- **Modules (JPMS)** stabilized post-9, but adoption has stayed slow industry-wide — most senior teams still treat modules as optional except for JDK internals encapsulation concerns.
- **Strong encapsulation of JDK internals by default (Java 17+, fully enforced by later releases)** — `sun.misc.Unsafe` and reflective access to internals increasingly restricted; libraries relying on deep reflection (older ORMs, some serialization frameworks) needed updates. This bit a lot of legacy codebases upgrading past 17.
- **Removal of deprecated tech:** Nashorn JS engine (removed Java 15), Applets (removed Java 17), Security Manager deprecated for removal (Java 17, actually disabled by default in later versions) — if you have ancient code depending on any of these, budget migration time.
- **`instanceof` pattern + switch pattern matching finalization** already covered above but worth flagging again as the single most impactful _daily_ coding style change.