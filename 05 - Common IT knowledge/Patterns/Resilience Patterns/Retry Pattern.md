**The idea:** Some failures are _transient_ — a blip, a momentary network hiccup. Just try again.

**But naively retrying is dangerous.** Two refinements are expected knowledge:

- **Exponential backoff** — don't retry instantly; wait progressively longer (100ms → 200ms → 400ms...). Otherwise you hammer an already-struggling service.
- **Jitter** — randomize that delay slightly. Why? If 1,000 clients all fail at the same moment and all retry with the exact same backoff schedule, they all hit the service again _simultaneously_ — a "thundering herd." Jitter spreads them out.

**Critical connection:** Retry is dangerous for **non-idempotent** operations. If "charge credit card" fails due to a network blip _after_ the charge actually succeeded server-side, retrying blindly double-charges the customer. This is why Retry and [[Idempotancy Key|Idempotancy]] are always discussed together.