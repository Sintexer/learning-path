Correct—the **lease expiration** makes it eligible for another worker to claim, provided **`next_attempt_at`** has also arrived. (The column was `next_attempt_at`, not `next_update_at`.)

- `lease_until`: “Is another worker still entitled to process this?”
- `next_attempt_at`: “Is it time to attempt this?”

Now let’s move to **2PC vs Saga, including when not to use Saga**.

# 2. Two-Phase Commit (2PC) vs Saga

## 1. What problem does 2PC solve?

Imagine transferring money between two separate databases:

```text
Database A: Debit Alice $100
Database B: Credit Bob $100
```

We want **atomicity**:

> Both changes commit, or neither does.

With ordinary independent transactions:

```text
A commits the debit
B fails before committing the credit
```

The databases disagree about the outcome.

**Two-Phase Commit** is a protocol that coordinates one commit-or-abort decision across multiple transactional participants.

## 2. Who participates?

There are two roles:

```text
Transaction coordinator
    ├── Database A
    └── Database B
```

- **Coordinator:** manages the overall decision.
- **Participants:** transactional resources that support the protocol.

An ordinary HTTP service does **not** automatically become a 2PC participant. Its underlying resource must support preparing and later committing or rolling back a transaction—often through mechanisms such as XA.

# 3. Phase one: prepare

The coordinator asks each participant:

> “Can you commit your part? Make yourself ready, but do not commit yet.”

```text
Coordinator                  Database A             Database B
     |                           |                      |
     |------ PREPARE ------------>|                      |
     |------ PREPARE ----------------------------------->|
     |                           |                      |
     |<----- YES ----------------|                      |
     |<----- YES ---------------------------------------|
```

Before voting **YES**, a participant must make its pending transaction durable enough to survive a crash and later follow the coordinator’s decision.

For typical relational database updates, this means retaining transaction state and relevant locks.

### Why is “YES” a serious promise?

After voting YES, the participant cannot safely decide on its own:

> “I waited too long, so I’ll roll back.”

The coordinator might already have decided to commit, and another participant might already have committed.

That is the source of 2PC’s famous **blocking problem**.

If a participant cannot prepare, it votes NO. The coordinator then decides to abort.

# 4. Phase two: commit or abort

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

If any participant votes NO—or preparation cannot complete—the coordinator can choose **ABORT** instead.

### Why save the decision first?

Suppose the coordinator:

```text
Tells A to commit
Crashes before telling B
```

After restarting, it must remember that the transaction’s decision was COMMIT and tell B the same thing.

It cannot change its mind after A has committed.

**Important:** 2PC does not make both databases commit at the exact same instant. It establishes a common durable decision and drives participants toward it.

# 5. What happens when the coordinator crashes?

Suppose both databases are prepared:

```text
A: PREPARED
B: PREPARED
Coordinator: unavailable
```

They may not know whether a commit decision was recorded.

While the decision cannot be established, they may have to remain prepared, holding resources and blocking conflicting operations.

Compare that with a Saga lease:

- A Saga worker’s expired lease allows another worker to repeat an **idempotent step**.
- A prepared 2PC participant cannot treat a timeout as permission to discard its transaction.

**Timeout does not resolve uncertainty in either design—but the recovery mechanisms differ.**

# 6. Why is 2PC usually a poor fit for microservices?

“2PC doesn’t scale” is interview shorthand. More precisely:

> 2PC adds coordination costs and availability coupling that become difficult across many independent services.

## A. Locks can outlive normal transaction times

A local database transaction might finish quickly.

A distributed transaction must wait for network communication and participating resources:

```text
Order DB is ready
Inventory DB is ready
Payment DB is slow
```

Prepared participants may hold locks while waiting.

Longer lock duration creates:

```text
More contention
    → More waiting
    → Longer transactions
    → Lower throughput
```

## B. Availability becomes coupled

Every required participant must be able to cooperate for the transaction to complete normally.

A slow or unavailable dependency can delay the whole transaction.

A Saga can instead persist:

```text
Order: PAYMENT_PENDING
```

and resume later without keeping the earlier database transactions open.

That does not make the order finish immediately—it lets the rest of the system continue doing useful work.

## C. Not all resources support it

Your workflow may include:

- A relational database
- An external payment provider
- An email service
- A shipping API

The payment provider will probably not expose:

```text
Prepare charge
Hold indefinitely
Commit or roll back on command
```

Email is even more obvious: once sent, it cannot participate in a normal transactional rollback.

## D. It increases operational coupling

Services must coordinate transaction protocols, recovery behavior, and resource configuration.

This clashes with the goal of independently operated services.

## E. Long business workflows are unsuitable

Some workflows last minutes, hours, or days:

```text
Create order
Wait for fraud review
Wait for customer approval
Arrange shipment
```

Keeping a distributed database transaction open for that duration is impractical.

# 7. How Saga differs

A Saga commits each local step immediately:

```text
Reserve inventory → COMMIT locally
Charge payment    → COMMIT locally
Confirm order     → COMMIT locally
```

If a later step fails, it performs new business actions:

```text
Refund payment
Release inventory
Cancel order
```

| Dimension | 2PC | Saga |
|---|---|---|
| Unit of coordination | One distributed transaction | Multiple local transactions |
| Failure handling | Commit or roll back | Retry or compensate |
| Earlier steps | Remain uncommitted until decision | Already committed |
| Resource holding | Can retain locks across coordination | No need to retain DB locks across workflow steps |
| Intermediate business states | Not exposed as separately committed steps before the decision | Explicitly visible |
| External APIs | Must support transaction participation | Can work with idempotent APIs and reconciliation |
| Long-running workflows | Poor fit | Good fit |
| Main application burden | Transaction infrastructure and recovery | Business states, compensation, idempotency |

**Nuance:** 2PC is a commit protocol, not by itself a guarantee of global serializable isolation. Isolation also depends on the databases and transaction system.

# 8. When would you NOT use Saga?

This is the important interview follow-up.

## Case A: One local database transaction is enough

Suppose both account balances live in the same database.

Use:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = :alice;

UPDATE accounts
SET balance = balance + 100
WHERE id = :bob;

COMMIT;
```

With appropriate balance checks and concurrency control, this is simpler and stronger than a Saga.

> Do not introduce distributed consistency problems when a local transaction can enforce the invariant.

## Case B: Intermediate inconsistency is unacceptable

Suppose the requirement truly is:

> “These records must change atomically. No separately committed intermediate state is acceptable.”

A Saga does not meet that requirement merely because it eventually compensates.

Options include:

- Put the data and invariant inside one transactional boundary.
- Use a database with suitable distributed transaction support.
- Use 2PC in a controlled environment where its trade-offs are acceptable.

Often this question reveals that the proposed service boundaries need reconsideration.

## Case C: Failure leaves an unacceptable, uncompensatable result

For example:

```text
Perform irreversible action A
Perform action B, which may fail
```

If A cannot be reversed or acceptably mitigated, Saga compensation cannot magically restore correctness.

You may need to redesign the workflow:

- Validate prerequisites first.
- Reserve resources before consuming them.
- Delay irreversible actions.
- Introduce manual approval or recovery.

Not every step in a Saga must be reversible, but the workflow needs a viable plan for failures around irreversible steps.

## Case D: A short transaction spans controlled, compatible resources

2PC can be reasonable when:

- All participants support it.
- Transactions are short.
- Infrastructure is tightly controlled.
- Atomicity is more important than availability during failures.
- The organization can operate the recovery machinery.

It is not inherently obsolete or always wrong.

# Interview-ready answer

> “2PC coordinates an atomic commit across transactional resources using prepare and commit phases. Prepared participants may retain locks and become blocked while the decision is unavailable, so it introduces latency, contention, and availability coupling. It is also unsuitable for many external APIs and long-running workflows. A Saga uses local commits, retries, and compensations instead, but exposes intermediate states and requires explicit business recovery. I would avoid Saga when a local transaction is sufficient or when strict atomicity is required and intermediate effects are unacceptable.”

### Quick check

**Both account balances are in the same database, but your application has separate `DebitService` and `CreditService` classes. Does having two service classes mean you need a Saga or 2PC?**