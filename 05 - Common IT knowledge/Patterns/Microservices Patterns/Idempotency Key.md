---
flashcards:
  q-wgut: { nid: 1795356721268, hash: eygrjx74, sync: b22e6iij }
  q-i5jh: { nid: 1795356721292, hash: bcmdzctz, sync: e3snamqd }
---

This pattern is about **client-facing APIs** (e.g., "place a bet" or "charge payment" [[REST]] endpoint; see [[API Idempotency]] and [[Idempotency]]). If a client's request times out and they [[Retry Pattern|retry]], how does the server avoid placing the bet twice?

**Pattern:** Client generates a unique key (usually a UUID) per logical operation, attaches it to the request (e.g., header `Idempotency-Key: abc-123`). Server, on receiving a request, checks: "have I seen this key before?" — typically stored in a database table (in the same [[Transaction]] as the business write) or [[Redis]] with a TTL (e.g., `idempotencyKey → result`, expires after 24h). If seen, return the cached result immediately without reprocessing. If not seen, process normally, then store the result under that key.

## Critical Edge Cases

1. **Concurrent duplicate requests (in-flight race)**: Two identical requests with the same key arrive milliseconds apart before the first finishes. Avoid [[Check-Then-Act]]! Insert the key atomically (`UNIQUE` constraint or Redis `SET NX`) with status `IN_PROGRESS`: the second request gets `409 Conflict` / `425 Too Early` or polls until the first finishes.
2. **Payload mismatch**: Client accidentally reuses an existing `Idempotency-Key` with a *different* request body (e.g., different amount). Hash the request payload alongside the key and reject mismatches with `422 Unprocessable Entity`.
3. **Storage choice — DB vs Redis**: Storing the idempotency record in [[Redis]] separately from a [[PostgreSQL]] business write reintroduces a [[Dual-Write]] window (DB commits, process crashes before Redis write → retry executes twice!). For financial/critical writes, store the idempotency key in the **same local DB transaction** as the business mutation.

> [!warning] Failure modes & trade-offs
> **TTL expiry**: Once an idempotency key expires (e.g., after 24h), a very late retry will execute as a brand-new operation. Choose TTL longer than the maximum possible client/queue retry window, and scope keys per `(tenant_id / user_id, endpoint, idempotency_key)` so one user cannot collide with another.

Idempotency Key:::A client-generated unique token attached to a mutating request so the server can deduplicate retries and return the original outcome instead of re-executing the operation.
^q-wgut

## Why is storing an Idempotency Key in Redis while writing business state to PostgreSQL risky? #card
It creates a [[Dual-Write]] failure window: if the PostgreSQL transaction commits and the process crashes before writing the key to Redis (or Redis evicts the key), a client retry will **execute the mutation a second time**. For critical writes, persist the idempotency key in the **same DB transaction** with a `UNIQUE` constraint.

How should a server handle a retry that arrives with the same `Idempotency-Key` while the first request is still in-flight?::Atomically claim the key (`INSERT ... ON CONFLICT` or `SET NX`) with an `IN_PROGRESS` state so the concurrent duplicate is rejected (e.g., `409 Conflict` / retry-later) rather than racing through [[Check-Then-Act]].
^q-i5jh

## Related
- [[API Idempotency]]
- [[Idempotency]]
- [[Idempotent Consumer]]
- [[Retry Pattern]]
- [[Check-Then-Act]]
- [[Dual-Write]]
- [[API Gateway]]
- [[Outbox Pattern]]
