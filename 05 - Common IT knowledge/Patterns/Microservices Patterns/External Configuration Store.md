**Purpose:** Separate operational configuration from application binaries.

Examples:
- Dependency URLs
- Timeouts
- Retry limits
- Feature flags
- Tenant-specific settings

Possible mechanisms include Spring Cloud Config, Consul, or platform-managed configuration such as Kubernetes ConfigMaps.

Secrets usually belong in a dedicated secret-management mechanism rather than ordinary configuration files.

### Why use it?

The same application artifact can run in different environments without rebuilding it.

Selected settings can also change without a full redeployment.

May also enable configuration versioning.

### Practical limitations

Dynamic configuration is a distributed change:


```
Instance A has version 10
Instance B still has version 9
```

Consequences can be serious if configuration changes business semantics.

Use:

- Validation.
- Versioning and audit history.
- Controlled rollout.
- Explicit refresh or restart behavior.
- A defined policy for store unavailability.

Many settings are read only at startup. External storage does not automatically make them dynamically refreshable.

For a long-running Saga, consider persisting the chosen policy version or deadline so an in-flight workflow does not silently change behavior halfway through.