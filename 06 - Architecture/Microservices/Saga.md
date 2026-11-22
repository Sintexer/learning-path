**What is the Saga Pattern?** The Saga pattern is a design pattern for managing multi-service business workflows without distributed locks ([[Database Per Service]] — compare with [[2PC vs Saga]]). It achieves [[Eventual Consistency]] across multiple services by breaking a workflow into a sequence of local [[ACID]] transactions, where each step has a corresponding **compensating action** (a new forward business transaction that semantically undoes the effect) if a later step fails.

## Types of Sagas (see [[Microservices Orchestration and Choreography]])

1. **Choreography-Based Saga**: Each service listens for domain events, executes its local transaction, and publishes its own events via the [[Outbox Pattern]]. No central coordinator — simple for 2–3 steps, but becomes hard to trace as workflows grow.
2. **Orchestration-Based Saga**: A central orchestrator (persisted in a [[Saga State Table]] with [[Worker Leases]], or built on [[Temporal]]) explicitly commands each step, tracks state, handles retries/timeouts, and drives compensations.

## Step Classification (Pivot Transaction Model)

Every step in a well-designed Saga falls into one of three categories:
1. **Compensatable transactions**: Steps *before* the pivot that can be semantically undone (e.g., `ReserveInventory`, `AuthorizePayment`).
2. **Pivot transaction**: The go/no-go point of no return — once it succeeds, the Saga commits to running to completion (e.g., `CapturePayment`).
3. **Retryable transactions**: Steps *after* the pivot that are guaranteed to eventually succeed via [[Retry Pattern|retries]] and [[Idempotency Key|idempotency]] (e.g., `ConfirmOrder`, `ScheduleShipment`).

## Example: E-Commerce Order Checkout

1. **Create Order** (`PENDING` — semantic lock).
2. **Reserve Inventory** (compensatable).
3. **Process Payment** (pivot).
4. **Ship Order** & **Confirm Order** (retryable).

If payment processing fails, compensating actions execute in reverse order:
1. **Cancel / Refund Payment** (if partial/ambiguous — verify via [[Reconciliation]]).
2. **Release Inventory**.
3. **Cancel Order**.

## Benefits

- **Availability & Scalability**: No cross-service database locks held across network hops (unlike [[Two-Phase Commit]]).
- **Resilience**: Each step retries independently, survives crashes, and works with external third-party APIs.

## Challenges

- **No Isolation (ACD, not ACID)**: Intermediate committed states are visible to other transactions.
- **Complexity**: Must design explicit state machines, idempotent handlers ([[Idempotent Consumer]]), and recovery paths.
- **Latency & Error Handling**: Sequential steps and compensations require durable tracking, monitoring, and [[Reconciliation]].

> [!warning] Failure modes & trade-offs
> - **Compensations are NOT atomic and CAN fail**: A compensation (`RefundPayment`, `ReleaseInventory`) is just another network call! It must be **idempotent**, **retried until it succeeds**, and escalated to [[Reconciliation]] / human ops if it permanently fails — you cannot "abort a rollback."
> - **Lack of Isolation (Dirty Reads & Lost Updates)**: While Order #42 is in `PAYMENT_PENDING`, another request could read or modify it. Use **semantic locks** (an explicit `PENDING` / `RESERVED` status flag checked by concurrent operations) and **commutative updates** where possible.
> - **Crash between DB update and command send**: Always persist Saga state transitions and outbound commands in the same transaction via the [[Outbox Pattern]] (or use [[Temporal]]).

Saga Pattern:::Managing a multi-service workflow as a sequence of local transactions where failures are handled by executing compensating business transactions instead of holding distributed locks.

## What are the three transaction types in a Saga (Compensatable, Pivot, Retryable)? #card
1. **Compensatable**: Steps before the pivot that can be semantically undone if a later step fails (e.g., reserve stock).
2. **Pivot**: The go/no-go point; once it succeeds, the Saga cannot roll back and must finish forward.
3. **Retryable**: Steps after the pivot that must eventually succeed via idempotent retries (e.g., confirm order, send receipt).

## Why can a Saga experience Isolation anomalies, and how do you mitigate them? #card
Because each step commits locally immediately, intermediate states are visible to concurrent transactions (no global Isolation). Mitigate with **semantic locks** (explicit `PENDING` / `RESERVED` flags), **commutative updates**, and ordering irreversible steps at or after the **pivot transaction**.

A Saga compensation is a ==new forward business transaction== (not a DB rollback), so it must be ==idempotent and retried until it succeeds==. #card-reminder

## Related
- [[2PC vs Saga]]
- [[Two-Phase Commit]]
- [[Microservices Orchestration and Choreography]]
- [[Temporal]]
- [[Saga State Table]]
- [[Worker Leases]]
- [[Outbox Pattern]]
- [[Idempotent Consumer]]
- [[Idempotency Key]]
- [[Reconciliation]]
- [[Database Per Service]]
- [[Eventual Consistency]]
