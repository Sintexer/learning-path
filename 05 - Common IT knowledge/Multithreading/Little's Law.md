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