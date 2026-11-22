---
flashcards:
  q-hnk7: { nid: 1795356722242, hash: 5zfic58z, sync: 8mxyjspp }
  q-cbp2: { nid: 1795356722266, hash: 2r8smimg, sync: yfvcctnx }
---

**Purpose:** Prevent another system's model from leaking into your domain (a core strategic [[DDD]] pattern).

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
- Data structures and schemas.
- Status meanings and state-machine enums.
- Error classifications (retryable vs terminal).
- Units, currencies, timezones, and identifiers.
- External workflow conventions.

### Why use it?

Changing providers or replacing a legacy system becomes less invasive — your core domain stays expressed in its own Ubiquitous Language rather than echoing a vendor's quirks.

### Practical limitation

Translation cannot erase semantic differences.

If the provider cannot distinguish "pending" from "unknown," the adapter must not fabricate certainty.

The layer should isolate foreign concepts — not accumulate unrelated domain rules.

> [!warning] Failure modes & trade-offs
> Two common failure modes:
> 1. **Leaky abstraction**: Passing raw vendor error codes or DTOs through "just for one field" gradually spreads the external model across your codebase.
> 2. **Bloated ACL**: Packing business logic (pricing, workflow retries, orchestration) into the translation layer turns it into an untestable second domain service. Keep the ACL strictly a **translator**.

This connects to [[Strangler Fig]], provider integration, and [[Reconciliation|payment reconciliation]].

Anti-Corruption Layer (ACL):::A boundary translation layer (from [[DDD]]) that maps an external or legacy system's data model, statuses, and errors into your own domain model so foreign concepts do not leak inward.
^q-hnk7

## What semantic mistake must an Anti-Corruption Layer avoid when translating external statuses? #card
Never **fabricate certainty** that the upstream system does not provide (e.g., mapping an ambiguous "unknown/timeout" state to "failed" or "pending"). Preserve explicit uncertainty so callers and [[Reconciliation]] jobs handle it safely.
^q-cbp2

## Related
- [[DDD]]
- [[Strangler Fig]]
- [[Reconciliation]]
- [[Overview]]
- [[Backward Compatibility]]
