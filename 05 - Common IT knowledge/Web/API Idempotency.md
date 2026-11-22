---
flashcards:
  q-jhty: { nid: 1795356719826, hash: w8vrn9zy, sync: gk27iyy3 }
  q-8587: { nid: 1795356719844, hash: t5y2eunk, sync: jqpptkwy }
  q-28iq: { nid: 1795356719866, hash: 8tafjjh6, sync: 53u86cni }
---

Based on request parameters and an [[Idempotency Key]], check if a request is a duplicate of an already executed operation so we avoid applying duplicated side effects. See [[Idempotency]] for the full cross-boundary model.

The problem:

```text
Client sends CreateOrder
Order Service commits O123
Response is lost
Client retries CreateOrder
```

Without protection, the service creates `O124`.

With a stable [[Idempotency Key]], the retry refers to `O123`.

In a **typical implementation** a table might contain:

```text
caller_id
operation_type
idempotency_key
request_hash
result_reference
```

A unique constraint protects the key within its scope:

```sql
UNIQUE (caller_id, operation_type, idempotency_key)
```

For a short database-only operation:

```text
One transaction:
    Claim idempotency key
    Create order
    Store order reference/result
    Commit
```

On retry:

```text
Find the existing operation
Verify request parameters match (compare request_hash)
Return its existing result
```

Concurrent duplicates may wait, receive an "in progress" response (`409 Conflict` or `425`), or retrieve the completed result, depending on the API contract.

### Important limitation

If the request performs an external payment, a local [[Transaction]] cannot make that payment atomic (see [[Dual-Write]]). Use a durable payment operation ([[Outbox Pattern]]) and provider idempotency, backed by [[Reconciliation]].

> [!warning] Failure modes & trade-offs
> - **Same key, different payload**: if a client accidentally reuses an idempotency key with a different amount or payload, returning the cached response silently corrupts business logic—verify `request_hash` and return `422` / `409`.
> - **Unbounded table growth vs retention window**: expiring keys too early (e.g., 1 hour) breaks delayed retries; keeping them forever bloats indexes—align TTL with client retry and [[DLQ]] replay windows (typically 24h–7d).
> - **5xx vs 4xx caching**: do *not* permanently lock a key on transient `5xx` server failures before side effects occurred, or the client can never retry with that key.

## Flashcards

What columns should an API idempotency table include to prevent cross-tenant collisions and payload-mismatch bugs?::`(caller_id, operation_type, idempotency_key)` with a `UNIQUE` constraint, plus `request_hash` and `result_reference` (or cached response).
^q-jhty

To avoid [[Dual-Write]] bugs, claiming the [[Idempotency Key]] and applying the local business state change must occur in the ==same database transaction==.
^q-8587

### How should an API handle a retry that reuses an existing idempotency key with a different request payload? #card

Compare the stored `request_hash` against the incoming payload and reject the request with `409 Conflict` or `422 Unprocessable Entity`—never silently return the cached result for a different operation.
^q-28iq

## Related
- [[Idempotency]]
- [[Idempotency Key]]
- [[Idempotent Consumer]]
- [[Retry Pattern]]
- [[Outbox Pattern]]
- [[Dual-Write]]
- [[Reconciliation]]
