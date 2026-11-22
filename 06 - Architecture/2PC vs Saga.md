---
flashcards:
  q-puyi: { nid: 1795356718041, hash: vsmx5h6a, sync: yd98prdz }
  q-dmrf: { nid: 1795356718066, hash: gz7nf5t7, sync: d825mg7t }
---

> [!note] Quick context on [[Worker Leases]]
> When recovering a [[Saga]] step, **lease expiration** makes a row eligible for another worker to claim, provided `next_attempt_at` has also arrived:
> - `lease_until`: "Is another worker still entitled to process this?"
> - `next_attempt_at`: "Is it time to attempt this?"

# Two-Phase Commit (2PC) vs. Saga

See also [[Two-Phase Commit]] and [[Saga]].

## 1. What problem does 2PC solve?

Imagine transferring money between two separate databases:

```text
Database A: Debit Alice $100
Database B: Credit Bob $100
```

We want **atomicity**:

> Both changes commit, or neither does.

With ordinary independent [[Transaction|transactions]]:

```text
A commits the debit
B fails before committing the credit
```

The databases disagree about the outcome.

**[[Two-Phase Commit]] (2PC)** is a protocol that coordinates one commit-or-abort decision across multiple transactional participants.

## 2. Who participates?

There are two roles:

```text
Transaction coordinator
    ├── Database A
    └── Database B
```

- **Coordinator:** manages the overall decision.
- **Participants:** transactional resources that support the protocol.

An ordinary [[HTTP]] service does **not** automatically become a 2PC participant. Its underlying resource must support preparing and later committing or rolling back a transaction — often through mechanisms such as XA.

## 3. Phase one: prepare

The coordinator asks each participant:

> "Can you commit your part? Make yourself ready, but do not commit yet."

```text
Coordinator                  Database A             Database B
     |                           |                      |
     |------ PREPARE ------------>|                      |
     |------ PREPARE ----------------------------------->|
     |                           |                      |
     |<----- YES ----------------|                      |
     |<----- YES ---------------------------------------|
```

Before voting **YES**, a participant must make its pending transaction durable enough to survive a crash and later follow the coordinator's decision.

For typical [[Relational Database|relational database]] updates, this means retaining transaction state and relevant [[Database Lock|locks]].

### Why is "YES" a serious promise?

After voting YES, the participant cannot safely decide on its own:

> "I waited too long, so I'll roll back."

The coordinator might already have decided to commit, and another participant might already have committed.

That is the source of 2PC's famous **blocking problem**.

If a participant cannot prepare, it votes NO. The coordinator then decides to abort.

## 4. Phase two: commit or abort

If everyone votes YES:

1. The coordinator durably records **COMMIT**.
2. It tells all participants to commit.

```text
Coordinator                  Database A             Database B
     |                           |                      |
     | Save COMMIT decision      |                      |
     |------ COMMIT ------------>|                      |
     |------ COMMIT ----------------------------------->|
     |                           |                      |
     |<----- ACK ----------------|                      |
     |<----- ACK ---------------------------------------|
```

If any participant votes NO — or preparation cannot complete — the coordinator chooses **ABORT** instead.

### Why save the decision first?

Suppose the coordinator:

```text
Tells A to commit
Crashes before telling B
```

After restarting, it must remember that the transaction's decision was COMMIT and tell B the same thing.

It cannot change its mind after A has committed.

**Important:** 2PC does not make both databases commit at the exact same instant. It establishes a common durable decision and drives participants toward it.

## 5. What happens when the coordinator crashes?

Suppose both databases are prepared:

```text
A: PREPARED
B: PREPARED
Coordinator: unavailable
```

They may not know whether a commit decision was recorded.

While the decision cannot be established, they must remain **in-doubt** (prepared), holding resources and blocking conflicting operations until the coordinator recovers.

Compare that with a [[Saga]] [[Worker Leases|lease]]:
- A Saga worker's expired lease allows another worker to repeat an **idempotent step** ([[Idempotency]]).
- A prepared 2PC participant cannot treat a timeout as permission to discard its transaction.

**Timeout does not resolve uncertainty in either design — but the recovery mechanisms differ.**

## 6. Why is 2PC usually a poor fit for microservices?

"2PC doesn't scale" is interview shorthand. More precisely:

> 2PC adds coordination costs and availability coupling that become difficult across many independent services ([[Database Per Service]]).

### A. Locks can outlive normal transaction times

A local database transaction might finish quickly. A distributed transaction must wait for network communication and participating resources:

```text
Order DB is ready
Inventory DB is ready
Payment DB is slow
```

Prepared participants hold locks while waiting. Longer lock duration creates:

```text
More contention → More waiting → Longer transactions → Lower throughput
```

### B. Availability becomes coupled

Every required participant must cooperate for the transaction to complete normally. A slow or unavailable dependency stalls the whole transaction.

A [[Saga]] can instead persist `Order: PAYMENT_PENDING` (in a [[Saga State Table]] or [[Temporal]]) and resume later without keeping earlier database transactions open.

### C. Not all resources support it

Your workflow may include a relational database, an external payment provider, an email service, and a shipping API. External APIs won't expose XA prepare/commit, and email cannot be rolled back once sent.

### D. It increases operational coupling

Services must coordinate transaction protocols, recovery behavior, and resource configuration — clashing with independently operated [[Microservices architecture|microservices]].

### E. Long business workflows are unsuitable

Workflows lasting minutes, hours, or days (fraud review, customer approval, shipment) cannot hold distributed database locks open.

## 7. How Saga differs

A [[Saga]] commits each local step immediately:

```text
Reserve inventory → COMMIT locally
Charge payment    → COMMIT locally
Confirm order     → COMMIT locally
```

If a later step fails, it performs compensating business actions:

```text
Refund payment
Release inventory
Cancel order
```

| Dimension | 2PC | Saga |
| --- | --- | --- |
| Unit of coordination | One distributed transaction | Multiple local transactions |
| Failure handling | Commit or roll back | Retry or compensate |
| Earlier steps | Remain uncommitted until decision | Already committed |
| Resource holding | Retains locks across coordination | No DB locks held across workflow steps |
| Intermediate business states | Hidden until commit decision | Explicitly visible (lack of Isolation) |
| External APIs | Must support transaction participation | Works with [[Idempotency Key\|idempotent APIs]] and [[Reconciliation]] |
| Long-running workflows | Poor fit | Good fit |
| Main application burden | Transaction infrastructure and recovery | Business states, compensation, idempotency |

**Nuance:** 2PC is an atomic commit protocol, not by itself a guarantee of global serializable isolation. Isolation also depends on the databases and concurrency control.

> [!warning] Failure modes & trade-offs
> In a **Saga**, you trade away **Isolation** (the "I" in [[ACID]]): other transactions can read or mutate intermediate committed state (e.g., dirty reads or lost updates before a compensation runs). Mitigate with **semantic locks** (explicit `PENDING` / `RESERVED` status flags), **pivot transactions** (ordering irreversible steps last), and **[[Reconciliation]]**.

## 8. When would you NOT use Saga?

### Case A: One local database transaction is enough

If both account balances live in the same database, use a single local [[ACID]] transaction (`BEGIN ... COMMIT`).

> Do not introduce distributed consistency problems when a local transaction can enforce the invariant.

### Case B: Intermediate inconsistency is unacceptable

If no separately committed intermediate state is acceptable, a Saga does not meet that requirement merely because it eventually compensates. Put the data inside one transactional boundary, use a distributed-SQL DB with consensus (e.g., [[Raft]]/[[Paxos]] + 2PC like Spanner/CockroachDB), or use 2PC in a controlled environment. Often this signals that service boundaries need reconsideration ([[Microservices decomposition]]).

### Case C: Failure leaves an unacceptable, uncompensatable result

If action A is irreversible and action B may fail, redesign the workflow: validate prerequisites first, reserve before consuming, delay irreversible actions (make them the **pivot** step), or introduce manual recovery.

### Case D: A short transaction spans controlled, compatible resources

2PC can be reasonable when all participants support it, transactions are short, infrastructure is tightly controlled, and atomicity outweighs availability during failures.

## Interview-ready answer

> "2PC coordinates an atomic commit across transactional resources using prepare and commit phases. Prepared participants may retain locks and become blocked while the decision is unavailable, so it introduces latency, contention, and availability coupling. It is also unsuitable for many external APIs and long-running workflows. A Saga uses local commits, retries, and compensations instead, but exposes intermediate states and requires explicit business recovery. I would avoid Saga when a local transaction is sufficient or when strict atomicity is required and intermediate effects are unacceptable."

### Quick check

**Both account balances are in the same database, but your application has separate `DebitService` and `CreditService` classes. Does having two service classes mean you need a Saga or 2PC?**
*(No — both classes share one database connection/transaction boundary, so a single local ACID transaction is sufficient.)*

## Why is Two-Phase Commit (2PC) called a blocking protocol? #card
Once a participant votes **YES** in the Prepare phase, it has promised to honor the coordinator's decision and **cannot unilaterally abort or commit**. If the coordinator crashes before delivering COMMIT/ABORT, prepared participants remain **in-doubt**, holding locks and blocking conflicting transactions until the coordinator recovers.
^q-puyi

## What ACID property does a Saga sacrifice compared to 2PC, and how do you mitigate it? #card
A [[Saga]] sacrifices **Isolation** because each local step commits immediately, making intermediate states visible to concurrent requests. Mitigate with **semantic locks** (e.g., `PAYMENT_PENDING` / `RESERVED` flags), ordering steps around a **pivot transaction**, and [[Reconciliation]].

When should you NOT use a Saga?::When a single local database transaction is sufficient, when intermediate states are strictly unacceptable, or when an irreversible step must execute before a failure-prone step without a viable mitigation.
^q-dmrf

## Related
- [[Two-Phase Commit]]
- [[Saga]]
- [[Saga State Table]]
- [[Worker Leases]]
- [[Temporal]]
- [[Outbox Pattern]]
- [[Reconciliation]]
- [[Database Per Service]]
- [[ACID]]
- [[Dual-Write]]
