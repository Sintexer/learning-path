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