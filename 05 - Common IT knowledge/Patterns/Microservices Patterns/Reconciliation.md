**Central question:** What happens when retries and event processing still leave an operation unresolved?

**Reconciliation** compares what our system believes with an authoritative source, then identifies and resolves discrepancies.

It is the operational safety net beneath asynchronous workflows.

Retry vs reconciliation:
- **Retry**: Attempt the operation again.
- **Reconciliation**: Establish what actually happened.