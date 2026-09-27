Temporal is a **durable execution platform**.

A durable execution can:
- Continue after its worker crashes.
- Wait without holding a thread for the whole waiting period.
- Retry failed external operations.
- Resume after receiving an external notification.
- Run for seconds, days, or longer.

Temporal is not merely:
- A message broker.
- A scheduled-job framework.
- A distributed transaction manager.

It coordinates durable processes through code. A [[Saga]] is one kind of process you can implement with it.

## The Four Main Concepts

### A. Workflow

A **Workflow** is the durable coordination logic.

For checkout:

```
Reserve inventory
Request payment
Wait for outcome
Confirm order or compensate
```

Workflow code decides **what should happen next**.

### B. Activity

An **Activity** performs work outside the workflow’s deterministic execution:

- Call Payment Service.
- Update a database.
- Reserve inventory.
- Send a notification.

Activity code is where normal Spring services, HTTP clients, and repositories usually belong.

### C. Worker

A **Worker** is your application process running Workflow and Activity code. It can be a Spring Boot application. Workers poll Temporal for tasks.

### D. Temporal Service

The Temporal Service manages:
- Durable execution history.
- Task dispatch.
- Timers.
- Retry scheduling.
- Workflow execution state.

It can be self-hosted or provided through Temporal Cloud.

#### Architecture

```
Your API
   |
   | Start checkout workflow
   v
Temporal Service
   |
   | Tasks through a Task Queue
   v
Your Worker application
   ├── Workflow code
   └── Activity code
           |
           ├── Inventory Service
           ├── Payment Service
           └── Order database/API
```

**Temporal does not host and run your Java business code for you.** Your workers execute it.

## What Problem Does It Solve?

A business workflow can outlive the process executing it:

```
Reserve inventory → Collect payment → Confirm order
```

Workers crash, dependencies time out, and some steps wait hours for approval.

Without a workflow platform, you implement:
 - Persisted workflow state.
- Workers that find unfinished work.
- Leases and concurrency protection.
- Retry scheduling and durable timers.
- Recovery logic and operational visibility.

**Temporal provides durable execution infrastructure for these processes.** You write the coordination logic; Temporal records execution history and manages scheduling and recovery.

> **Interview definition:** Temporal is a durable execution platform that lets workflows survive worker failures and long waits by recording execution history and replaying workflow logic.

## The Mental Model

|Concept|Responsibility|
|---|---|
|**Workflow**|Decides what should happen next|
|**Activity**|Performs external work: HTTP calls, database writes, payments|
|**Worker**|Your application process executing Workflow and Activity code|
|**Temporal Service**|Stores history and manages tasks, timers, and execution progress|
|**Task Queue**|Routes execution tasks to compatible workers|

Temporal does not run your Java business code inside its server. **Your workers execute it.**

### Workflow replay versus Activity retry

These are different mechanisms:
- **Workflow replay:** reconstructs progress using recorded history. Completed Activity results are reused.
- **Activity retry:** executes external work again because a successful completion was not recorded or an attempt failed.

This distinction explains both Temporal’s durability and its need for idempotency.

## What Becomes Unnecessary—and What Does Not?

### Usually replaced for workflows managed by Temporal

- Custom Saga polling loops.
- SQL leases for assigning workflow steps.
- Hand-built retry schedulers.
- Timer tables for delayed workflow execution.
- Manual persistence of every execution transition.
- Much of the custom workflow-recovery infrastructure.

These are not universally obsolete; Temporal takes over these responsibilities **within its workflow execution model**.

### Still required

| Pattern or concern                              | Why it remains                                                                          |
| ----------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Saga compensation**                           | Temporal cannot decide whether your business needs a refund                             |
| **Idempotency**                                 | Activities can execute more than once                                                   |
| **Reconciliation**                              | A timed-out external operation may have succeeded                                       |
| **Reservations and concurrency control**        | Durable workflows do not provide global isolation                                       |
| **Outbox**                                      | Database writes and external publication may still be separate commits                  |
| **Circuit Breaker / rate limiting / bulkheads** | Retries alone do not protect dependencies from overload                                 |
| **Domain persistence and CQRS projections**     | Workflow history is not automatically the application’s business database or read model |

### Saga Orchestration

Temporal is a natural implementation platform for an **orchestrated [[Saga]]**.

```
Workflow: coordinates checkout
Activities: invoke Inventory and Payment services
Compensations: release inventory, refund payment
```

It supplies execution durability, not business correctness.

**Compensation is still a new business operation—not a database rollback.**

### Event Sourcing

Temporal uses an event history to recover workflow execution.

That does **not** mean your business domain uses [[Event Sourcing]].
- Temporal history describes workflow execution.
- Domain Event Sourcing treats business events as the authoritative representation of domain state.

They can coexist, but neither implies the other.

### Message Brokers

Temporal task queues distribute execution tasks to workers. They are not a general replacement for Kafka topics used for:
- Broadcasting domain events.
- Analytics pipelines.
- Independent consumer groups.
- Long-term event replay.

### Outbox

This remains a [[Dual-Write]] problem:

```
Commit order to PostgreSQL
Start Temporal Workflow
```

A crash between the two can leave an order without a workflow.

Possible designs include:

- Persist a start-workflow intent in an outbox, then start using a stable Workflow ID.
- Start the Workflow first and let an idempotent Activity create the order.

Temporal does not automatically make its own state atomic with your database.

## The Most Important Interview Traps

### 1. “Temporal guarantees exactly-once payment.”

**False.**

```
Provider charges customer
Worker crashes before reporting Activity completion
Temporal retries Activity
```

Use the same payment operation ID across retries, provider idempotency, and status reconciliation. Temporal cannot atomically commit an external charge together with its execution history.

### 2. “Workflow replay repeats every external call.”

**False.**

Successfully recorded Activity results are supplied from history during replay. However, an Activity whose completion was not recorded may run again.

### 3. “I can write arbitrary Java inside a Workflow.”

**False.**

Workflow decisions must be replay-compatible. Direct network calls, database reads, uncontrolled randomness, or changing wall-clock inputs can make replay diverge.

External work belongs in Activities. Use Temporal’s supported APIs for durable waits and replay-safe values.

### 4. “Temporal automatically compensates failed Sagas.”

**False.**

You define:

- Which effects need compensation.
- When cancellation becomes irreversible.
- How late successes are handled.
- What to do when compensation fails.

Do not assume ordinary Workflow cancellation automatically completes cleanup.

### 5. “Deploying changed Workflow code is just like changing a REST handler.”

**Not always.**

Existing workflows may replay histories produced by older code.

Changes must preserve compatibility or use appropriate Workflow versioning and worker-deployment mechanisms.

### 6. “Retries are always helpful.”

**False.**

Unbounded retries can overload an unhealthy dependency or keep a business process alive past its deadline.

Distinguish:

- Technical failures worth retrying.
- Business rejections that require another path.
- Unknown outcomes that require reconciliation.

## Practical Limitations

Temporal trades custom coordination code for a platform and programming model:

- You must operate Temporal or use a managed offering.
- Workflow code has determinism and evolution constraints.
- Large or long-lived execution histories require management, such as **Continue-As-New**.
- History contains Activity inputs and outputs, so sensitive data needs careful handling.
- Operational visibility does not replace business alerts for stuck refunds or unresolved payments.

If the Temporal Service is unavailable, new scheduling and recorded progress can be delayed. Already-running external work may still finish, so idempotency remains essential.

## When Would You Use It?

**Good fit:**

- Multi-step workflows across services.
- Long-running approvals or fulfillment.
- Durable timers and callbacks.
- Complex retries and compensation.
- Workflows where operational recovery is expensive to implement manually.

**Possibly unnecessary:**

- Simple CRUD inside one database transaction.
- A small scheduled task with limited recovery requirements.
- Straightforward event consumers without substantial coordination.

> Choose Temporal when durable coordination is a significant problem—not merely because the system has microservices.

## Interview Q&A

### Q: Is Temporal a distributed transaction manager?

**A:** No. It coordinates durable execution but does not provide one atomic commit across service databases and external APIs.

### Q: What happens when a worker crashes?

**A:** Temporal can dispatch further tasks to another compatible worker. Workflow progress is reconstructed from history; unfinished Activities may be retried according to their timeout and retry configuration.

### Q: Does a workflow need to hold a thread while waiting for a day?

**A:** No. Durable timers and waits allow execution to suspend without dedicating an operating-system thread for that whole period.

### Q: Do I still need an order status in my database?

**A:** Often yes. The application needs authoritative business records and convenient customer-facing queries. Temporal manages execution progress; the boundary between workflow state and domain state must be designed explicitly.

### Q: How do external notifications reach a workflow?

**A:** Through mechanisms such as **Signals** or **Updates**. The workflow then evaluates the notification against its current state. Duplicate or stale business notifications still need appropriate handling.

### Q: How does Temporal handle a late payment success after cancellation?

**A:** It durably executes the logic you define: record the successful charge, preserve cancellation intent, initiate an idempotent refund, and wait or reconcile until cleanup completes.

### Q: What is the strongest concise answer about its value?

**A:** “Temporal replaces much of the infrastructure needed to run recoverable workflows—state recovery, task scheduling, retries, and timers. It does not replace business state machines, compensation design, idempotency, or external-system reconciliation.”