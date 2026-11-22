**Central question:** What happens when [[Retry Pattern|retries]], [[Outbox Pattern|outboxes]], and event processing still leave an operation unresolved or drifted?

**Reconciliation** compares what our system believes with an authoritative source, then identifies and resolves discrepancies.

It is the operational safety net beneath asynchronous workflows, [[Saga|Sagas]], and third-party integrations.

**Retry vs. reconciliation:**
- **[[Retry Pattern|Retry]]**: Attempt the operation again.
- **Reconciliation**: Establish what actually happened and converge state toward truth.

## Where Reconciliation Is Essential

1. **Payment / third-party settlement**: A payment call times out after leaving our network — did the PSP capture the card or drop it? A reconciliation job queries the PSP's status/settlement API (or ingests daily settlement files) and converges local state (`PAYMENT_UNKNOWN` → `CAPTURED` or `FAILED`).
2. **Cross-service [[CQRS]] projections**: Missed events, bugs in an older projection version, or [[DLQ]] drops can cause a read model to drift from the write store. Periodic reconciliation checksums or re-syncs drifted entities.
3. **Stuck [[Saga|Sagas]] / expired [[Worker Leases]]**: A sweeper scans for workflows stuck in `PENDING` past a threshold, queries downstream services for ground truth, and drives them to completion or compensation.
4. **[[Strangler Fig]] migrations**: Comparing legacy DB records against the new service's DB (similar to [[Shadow Testing]]) to prove parity before cutting over.

## Design Principles

- **Idempotent repair**: Running the reconciliation job twice must be safe (re-use [[Idempotency Key]] / [[Version Guard]]).
- **Grace window**: Only reconcile records older than normal eventual-consistency lag (e.g., `updated_at < now() - interval '5 minutes'`) so you don't flag in-flight operations as broken.
- **Categorize discrepancies**: Auto-heal safe/known drifts (e.g., missed projection update); **escalate to humans** for financial anomalies (e.g., money captured at PSP for a canceled order).
- **Measure drift rate**: Treat rising reconciliation fix counts as a **bug alert**, not "business as usual" — reconciliation is a safety net, not a substitute for fixing broken pipelines.

> [!warning] Failure modes & trade-offs
> **Masking silent bugs**: If a reconciliation job quietly repairs 10,000 drifted rows every night without alerting, teams won't notice that a primary event pipeline has been broken for weeks. Always emit metrics (`reconciliation_discrepancies_found`, `reconciliation_auto_repaired`, `reconciliation_escalated`) and page when drift exceeds a tiny baseline.

Reconciliation:::A periodic or triggered process that compares local state against an authoritative source of truth to detect, repair, or escalate discrepancies left by crashes, timeouts, or missed events.

## What is the difference between Retry and Reconciliation? #card
- **[[Retry Pattern|Retry]]** re-attempts an operation assuming it failed transiently.
- **Reconciliation** queries an authoritative source to **establish what actually happened** and converges drifted or ambiguous state (e.g., an unknown payment timeout or drifted [[CQRS]] projection).

Always apply a ==grace window== (ignore recently updated records) in reconciliation jobs so you do not treat normal ==in-flight eventual consistency== as corruption. #card-reminder

## Related
- [[Saga]]
- [[Outbox Pattern]]
- [[CQRS]]
- [[DLQ]]
- [[Anti-Corruption Layer]]
- [[Strangler Fig]]
- [[Shadow Testing]]
- [[Anti-Entropy]]
- [[Dual-Write]]
