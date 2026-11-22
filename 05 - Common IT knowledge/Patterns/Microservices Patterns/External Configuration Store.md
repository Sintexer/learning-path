---
flashcards:
  q-gqwa: { nid: 1795356721892, hash: tdrav2e9, sync: jhcgjd94 }
  q-abkf: { nid: 1795356721925, hash: dgzikrwp, sync: yatcr8jr }
---

**Purpose:** Separate operational configuration from application binaries (a Twelve-Factor / [[Cloud-Native App]] principle).

Examples:
- Dependency URLs
- [[Timeout Pattern|Timeouts]]
- [[Retry Pattern|Retry limits]]
- Feature flags
- Tenant-specific settings

Possible mechanisms include Spring Cloud Config, Consul, [[etcd]], or platform-managed configuration such as [[Kubernetes]] ConfigMaps.

Secrets usually belong in a dedicated secret-management mechanism (e.g., Vault, AWS Secrets Manager, KMS) rather than ordinary configuration files.

### Why use it?

The same application artifact can run in different environments without rebuilding it.

Selected settings can also change without a full redeployment.

May also enable configuration versioning and centralized auditability.

### Practical limitations

Dynamic configuration is a distributed change:

```
Instance A has version 10
Instance B still has version 9
```

Consequences can be serious if configuration changes business semantics.

Use:
- Schema/bounds validation before publishing.
- Versioning, rollback, and audit history.
- Controlled / canary rollout.
- Explicit refresh or restart behavior.
- A defined policy for store unavailability (cached last-known-good config so restarts don't fail if the config server is down).

Many settings (like thread-pool sizes or DB connection pool limits) are read only at startup. External storage does not automatically make them dynamically refreshable.

For a long-running [[Saga]], consider persisting the chosen policy version or deadline so an in-flight workflow does not silently change behavior halfway through.

> [!warning] Failure modes & trade-offs
> **Config-induced global outage**: Unlike a rolling code deploy, a bad dynamic config push can hit 100% of instances simultaneously and take down the whole fleet in seconds. Treat config changes with the same rigor as code: validate schemas, roll out progressively (canary/percentage), and keep an automatic kill-switch/rollback.

## Why is a dynamic configuration push often more dangerous than a code deployment? #card
Code deploys roll out gradually (canary/rolling), whereas a dynamic config update can propagate to **100% of instances simultaneously** and bypass CI tests — instantly turning a typo or bad threshold into a fleet-wide outage. Mitigate with schema validation, staged rollouts, and cached last-known-good fallbacks.
^q-gqwa

For long-running [[Saga|Sagas]], persist the ==policy version or deadline== when the workflow starts so dynamic config updates do not alter in-flight executions mid-stream. #card-reminder
^q-abkf

## Related
- [[Cloud-Native App]]
- [[Kubernetes]]
- [[etcd]]
- [[Zero-downtime deployment]]
- [[Saga]]
- [[Health Checks]]
