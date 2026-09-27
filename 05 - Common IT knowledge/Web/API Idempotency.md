Based on request parameters, check if request is a duplication of an already executed operation. Avoid submitting duplicated side effect.

The problem:

```text
Client sends CreateOrder
Order Service commits O123
Response is lost
Client retries CreateOrder
```

Without protection, the service creates O124.

With a stable request key, the retry refers to O123.

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
Verify request parameters match
Return its existing result
```

Concurrent duplicates may wait, receive an “in progress” response, or retrieve the completed result, depending on the API contract.

### Important limitation

If the request performs an external payment, a local transaction cannot make that payment atomic. Use a durable payment operation and provider idempotency.
