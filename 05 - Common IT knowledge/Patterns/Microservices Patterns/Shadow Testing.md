Shadow testing (also known as **traffic shadowing** or **traffic mirroring**) is a production testing technique where a copy of live production traffic is duplicated and sent to a new preview version of an application in the background.

> [!tip] AKA
> - Shadow Traffic / Dark Launching (when hidden behind flags)

The key characteristic of shadow testing is that the new version processes these real-world requests "in the shadow." Its responses are discarded and never sent back to the actual users, ensuring that the new code has zero impact on the live user experience.

## How Shadow Testing Works

1. **User Request:** A real user triggers an action (e.g., searches for a product on an e-commerce site).
2. **Traffic Splitting:** A [[Reverse proxy|proxy]], [[Load Balancing|load balancer]], [[API Gateway]], or [[Service Mesh]] (like Envoy or Istio — see [[Microservices Service mesh pattern]]) asynchronously duplicates the incoming request.
3. **Primary Production:** The original request goes to the current live version (V1). The user gets their response immediately.
4. **Shadow Environment:** The cloned request is sent to the new version (V2). V2 processes the request in isolation.
5. **Comparison & Analytics:** Engineers compare performance, error rates, and response diffs of both V1 and V2 to identify bugs or latency regressions.

## Key Benefits

- **Real-World Load Testing:** Tests how the new system handles actual production scale, peak traffic, weird payloads, and unpredictable human behavior — far more realistic than staging.
- **Zero User Risk:** If the new version crashes, errors out, or experiences severe latency, the end-user remains unaffected because they only see the output from V1.
- **Data & Logic Validation:** Verifies that the new version generates the exact same business logic results as the old version before cutover (pairs well with [[Strangler Fig]]).

## The Challenge of Side-Effects (Read vs. Write)

Shadow testing is straightforward for **Read** operations (e.g., viewing a profile, searching a catalog) because fetching data doesn't change the state of the system.

However, it requires careful engineering for **Write** operations (e.g., placing an order, modifying a database, sending an email). If a write request is blindly duplicated, the shadow version might charge a customer's credit card a second time or create duplicate records. To solve this, developers typically stub/virtualize third-party APIs, suppress outbound notifications/events, or route shadow traffic to a completely isolated, mirrored database.

> [!warning] Failure modes & trade-offs
> - **Hidden production impact**: Even when responses are discarded, if V2 shares a **database, cache ([[Redis]]), or rate-limited third-party API** with V1, shadow traffic **doubles the load** on shared infrastructure and can degrade V1!
> - **Non-deterministic diffs**: Timestamps, UUIDs, floating-point ordering, or [[Eventual Consistency|eventual consistency]] lag between V1 and V2 databases create noisy false-positive diffs unless normalized in the comparator.
> - **PII & compliance**: Mirroring live traffic copies production PII into the shadow environment, which must meet the same security and retention controls.

Shadow Testing (Traffic Mirroring):::Duplicating live production requests asynchronously to a preview/new service version while discarding its responses, validating real-world performance and output parity with zero user-facing impact.

## What two hidden dangers must you guard against when running Shadow Testing on production traffic? #card
1. **Duplicate side effects on writes**: Blindly mirroring writes can double-charge users, send duplicate emails, or corrupt shared state unless outbound calls and storage are isolated/stubbed.
2. **Shared infrastructure overload**: Even for read-only endpoints, if V2 hits the same database or cache as V1, mirroring **doubles load** on shared dependencies and can degrade live V1 traffic.

## Related
- [[Strangler Fig]]
- [[Zero-downtime deployment]]
- [[Microservices Service mesh pattern]]
- [[Service Mesh]]
- [[API Gateway]]
- [[Reconciliation]]
