**The idea:** When a call fails (or the circuit is open), don't just propagate an error — do something graceful instead.

**Have this example ready, verbatim style:**

> "If the live odds service is down, fall back to the last cached odds with a 'may be outdated' flag, rather than showing an error to the user."

That's a strong answer because it's concrete and shows product thinking, not just technical mechanism.