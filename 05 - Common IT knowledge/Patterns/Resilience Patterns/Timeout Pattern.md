Simple, but people forget to mention it — and it's foundational to everything else.

**The rule:** Never let a remote call wait indefinitely. An unbounded wait means an unbounded hold on a thread — which is exactly what causes the thread pool exhaustion problem Circuit Breaker exists to prevent. Timeout is the _first line of defense_; Circuit Breaker is what happens after repeated timeouts.