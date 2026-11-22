**The idea:** Some failures are _transient_ — a blip, a momentary network hiccup, a pod rolling over. Just try again.

**But naively retrying is dangerous.** Two refinements are expected knowledge:

- **Exponential backoff** — don't retry instantly; wait progressively longer (100ms → 200ms → 400ms...). Otherwise you hammer an already-struggling service.
- **Jitter** — randomize that delay (e.g., Full Jitter: `random(0, min(cap, base * 2^attempt))`). Why? If 1,000 clients all fail at the same moment and all retry with the exact same backoff schedule, they all hit the service again _simultaneously_ — a "thundering herd." Jitter spreads them out.

**Critical connection:** Retry is dangerous for **non-idempotent** operations. If "charge credit card" fails due to a network blip _after_ the charge actually succeeded server-side, retrying blindly double-charges the customer. This is why Retry, [[Idempotency]], [[API Idempotency]], and [[Idempotency Key]] are always discussed together.

> [!warning] Failure modes & trade-offs
> - **Retry amplification (retry storms)**: If Service A retries 3×, Service B retries 3×, and Service C retries 3×, a single failing request fans out into \(3^3 = 27\) calls at the leaf — turning a minor brownout into a self-inflicted DDoS. **Fix:** retry at **only one layer** (typically the outermost caller or leaf edge), enforce a **retry budget** (e.g., max 10–20% of total traffic may be retries), and respect overall request deadlines ([[Timeout Pattern]]).
> - **Retrying non-transient errors**: Never retry `400 Bad Request`, `401/403`, `404`, or business-rule rejections — only retry transient network errors, `429` (honoring `Retry-After`), and `502/503/504`.
> - **Retries vs [[Circuit Breaker]]**: Wrap Retry **inside** or **outside** the Circuit Breaker intentionally — if the circuit is already Open, stop retrying immediately so you don't burn threads waiting on backoff sleeps.

## Why do multi-layer retries cause retry storms, and how do you prevent them? #card
Retries multiply across hops: 3 retries at each of 3 layers turns 1 failing request into \(3^3 = 27\) downstream calls, crushing a struggling service. Prevent this by:
1. Retrying at **one designated layer** only.
2. Using **exponential backoff with jitter**.
3. Enforcing a **retry budget** (cap retries as a small % of total traffic) and honoring an end-to-end deadline.

Only retry operations that are ==idempotent== (or guarded by an [[Idempotency Key]]) and only on ==transient failures== (timeouts, network resets, 429/502/503/504). #card-reminder

What problem does jitter solve on top of exponential backoff?::Prevents synchronized thundering herds when many clients fail at the same instant and would otherwise wake up and retry at identical intervals.

## Related
- [[Microservices Resilience and Fault tolerance]]
- [[Timeout Pattern]]
- [[Circuit Breaker]]
- [[Bulkhead Pattern]]
- [[Idempotency Key]]
- [[API Idempotency]]
- [[Backpressure]]
- [[Rate Limiting]]
