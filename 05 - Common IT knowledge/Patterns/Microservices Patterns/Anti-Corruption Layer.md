**Purpose:** Prevent another system’s model from leaking into your domain.

```
External or legacy API
          |
          v
Anti-Corruption Layer
          |
          v
Your domain model
```

Example:

```
Provider: status = "SETTLED"
Your model: PaymentStatus.CAPTURED
```

The adapter translates:

- Data structures.
- Status meanings.
- Error classifications.
- Units and identifiers.
- External workflow conventions.

### Why use it?

Changing providers or replacing a legacy system becomes less invasive.

### Practical limitation

Translation cannot erase semantic differences.

If the provider cannot distinguish “pending” from “unknown,” the adapter must not fabricate certainty.

The layer should isolate foreign concepts—not accumulate unrelated domain rules.

This connects to [[Strangler Fig]], provider integration, and payment reconciliation.