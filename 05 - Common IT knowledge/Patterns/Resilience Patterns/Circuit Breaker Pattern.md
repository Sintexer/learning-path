> Prevents cascading failures

**The problem it solves:** Imagine Service A calls Service B, and B is dying — every call to B takes 30 seconds to time out. If A keeps calling B anyway, A's threads pile up waiting on B, A's thread pool exhausts, and now A itself becomes unresponsive to _its_ callers. One failing service cascades upward and takes down the whole chain.

**The fix:** Stop calling the broken thing. Fail fast instead.

**Three states:**

| State         | Behavior                                                                                                        |
| ------------- | --------------------------------------------------------------------------------------------------------------- |
| **Closed**    | Normal — calls go through, failures are counted                                                                 |
| **Open**      | Threshold exceeded — calls short-circuit immediately, no attempt made, usually returns fallback/error instantly |
| **Half-Open** | After a cooldown, a few test calls are let through. Success -> back to Closed. Failure -> back to Open.         |

Think of it like an electrical circuit breaker in your house — trips when there's too much current (too many failures), and you have to manually (or here, automatically after a timeout) test if it's safe to reset.

**Libraries:** Resilience4j (the modern standard in the Java/Kotlin/Spring world — Netflix Hystrix is the older one, now in maintenance mode). If someone asks "how would you implement this in Spring," that's your answer.