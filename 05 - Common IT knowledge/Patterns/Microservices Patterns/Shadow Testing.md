Shadow testing (also known as traffic shadowing or traffic mirroring) is a software testing production technique where a copy of live production traffic is duplicated and sent to a new preview version of an application in the background.

> [!tip] AKA
> - Shadow Traffic

The key characteristic of shadow testing is that the new version processes these real-world requests "in the shadow." Its responses are discarded and never sent back to the actual users, ensuring that the new code has zero impact on the live user experience.

## How Shadow Testing Works

1. User Request: A real user triggers an action (e.g., searches for a product on an e-commerce site).
2. Traffic Splitting: A proxy, load balancer, or service mesh (like Envoy or Istio) duplicates the incoming request.
3. Primary Production: The original request goes to the current live version (V1). The user gets their response immediately.
4. Shadow Environment: The cloned request is sent to the new version (V2). V2 processes the request in isolation.
5. Comparison & Analytics: Engineers compare the performance, error rates, and outputs of both V1 and V2 to identify bugs or performance regressions.

## Key Benefits

- Real-World Load Testing: It tests how the new system handles actual production scale, peak traffic, and unpredictable human behavior, which is incredibly difficult to simulate in standard staging environments.
- Zero User Risk: If the new version crashes, errors out, or experiences severe latency, the end-user remains completely unaffected because they only see the output from the stable, live version.
- Data & Logic Validation: It allows developers to verify that the new version generates the exact same correct business logic results as the old version before fully migrating.

## The Challenge of Side-Effects (Read vs. Write)

Shadow testing is straightforward for Read operations (e.g., viewing a profile, searching a catalog) because fetching data doesn't change the state of the system.

However, it requires careful engineering for Write operations (e.g., placing an order, modifying a database, sending an email). If a write request is blindly duplicated, the shadow version might charge a customer's credit card a second time or create duplicate records. To solve this, developers typically use virtualization/mocking for third-party APIs or route the shadow traffic to a completely isolated, mirrored database.