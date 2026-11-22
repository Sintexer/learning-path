---
flashcards:
  q-y4a4: { nid: 1795356718342, hash: 6w9m4imf, sync: eun7hicg }
  q-gfbf: { nid: 1795356718367, hash: siidqrtw, sync: 5ckxxi56 }
  q-ce9x: { nid: 1795356718392, hash: awq5w578, sync: j7uui82p }
---

At its simplest, **Little’s Law** is a formula used to calculate how long it takes for something to get through a system (like a store, a factory, or a software project).

It states that the number of items in a system is equal to the rate at which they arrive multiplied by the time they spend in the system.

### The Formula

$L = λ × W$

*   **L (Inventory/Work in Progress):** The average number of items in the system.
*   **λ (Throughput/Arrival Rate):** How many items enter the system per unit of time.
*   **W (Wait Time/Cycle Time):** How long, on average, an item spends in the system.

> [!example] A Simple Example: The Coffee Shop
> Imagine you own a coffee shop.
> 
> 1.  **Arrival Rate (λ):** You notice that **10 customers** walk in every hour.
> 2.  **Wait Time (W):** You notice that, on average, a customer spends **30 minutes (0.5 hours)** in your shop from the moment they order until they leave.
> 
> **Using Little’s Law:**
> L = 10 (customers/hour) × 0.5 (hours) = 5
> 
> This means that, at any given moment, you should expect to see **5 customers** inside your shop.

### Why is this useful?

Little’s Law is powerful because it shows the **mathematical relationship** between speed and volume. If you want to change one, you have to change the others:

*   **If you want to reduce the "Wait Time" (W):** You either have to reduce the number of people in the shop (L) or increase the speed at which you serve them (λ).
*   **If you have a backlog (L):** You can see exactly how much faster you need to work (λ) to get the wait time (W) down to a specific goal.

### The "Golden Rule" of Little's Law

The most important takeaway is this: **If you want to get things done faster (reduce W), you must reduce the amount of work in progress (L).**

If you have too many projects on your desk at once (high L), your wait time for any single project will inevitably go up, even if your speed (λ) stays the same.

> [!tip] Production reality
> - **Sizing thread and connection pools:** If a service handles `λ = 500 req/s` and each request holds a worker thread for `W = 0.04 s` (40 ms), you need at least `L = 500 × 0.04 = 20` active threads to sustain throughput—before adding headroom for latency spikes or tail latency.
> - **Stability precondition:** Little's Law only holds for a **stable** system (arrival rate ≤ max service rate, i.e. utilization < 100%). Once `λ` exceeds capacity, `L` and `W` grow without bound—you must apply [[Backpressure]], [[Rate Limiting]], or load shedding rather than enlarging queues.
> - **Queueing vs. concurrency:** Increasing queue size without increasing service rate only inflates `L` and therefore `W` (higher latency), risking client [[Timeout Pattern|timeouts]] and retry storms.

## Flashcards

Little's Law relates average concurrency, arrival rate, and latency as ==L = λ × W== (average items in system = arrival rate × average time in system).
^q-y4a4

## When does Little's Law break down in production systems, and how do you protect the system? #card
Little's Law assumes a **stable** system where arrival rate (`λ`) does not exceed service capacity. When `λ` exceeds capacity, in-flight work (`L`) and wait time (`W`) grow unboundedly until OOM or cascading [[Timeout Pattern|timeouts]]. Protect the system with bounded queues, [[Backpressure]], [[Rate Limiting]], and load shedding.
^q-gfbf

Adding a bigger queue without increasing processing throughput only increases `L` and worsens latency `W`—bound queues and shed load early. #card-reminder
^q-ce9x

## Related
- [[Mutual Exclusion]]
- [[IO-Bound vs CPU-Bound]]
- [[Backpressure]]
- [[Rate Limiting]]
- [[Bulkhead Pattern]]
- [[Timeout Pattern]]