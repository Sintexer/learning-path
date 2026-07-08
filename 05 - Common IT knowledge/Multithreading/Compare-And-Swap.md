---
aliases:
  - CAS
---
**CAS** stands for **Compare-And-Swap**. It is a way to update a piece of data without using "locks" (which can be slow and cause programs to freeze). It is a concrete technology that makes [[Optimistic Concurrency Control]] possible.

> [!tip]
> **CAS is not a system call, and it is not implemented by the OS.** It is implemented by the **CPU hardware itself.**

Think of it like a **"conditional update"** based on a promise.

### How it works (The 3-Step Process)

Imagine you want to update a value (e.g., a bank balance) from **100 to 150**.

1.  **Read:** You look at the current value. You see it is **100**.
2.  **Compare:** You ask the system: *"Is the value still 100?"*
3.  **Swap:** 
	*   **If YES:** You change it to **150**. (Success!)
	*   **If NO:** Someone else changed it while you were looking. You don't update it; instead, you start over and try again.

### Why use it?

*   **No Locking:** You don't have to "lock" the data and make other threads wait. You just try to update it, and if someone else got there first, you just try again.
*   **Efficiency:** It is much faster than traditional locking because it avoids the "stop-and-wait" overhead.

**In one sentence:** CAS is a way of saying, *"Update this value only if it hasn't changed since I last looked at it."*