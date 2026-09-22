**Purpose:** Reuse code across services without adding a network dependency.

Good candidates include:
- Logging and tracing setup.
- Small security integration helpers.
- Standard client instrumentation.
- Carefully scoped technical utilities.

### Why use it?

A library offers local execution and consistent implementation of repeated technical concerns.

### Practical limitations

A library duplicates its code into every consuming deployment.

Therefore:
- Fixes require upgrading and redeploying consumers.
- Different services may run different versions.
- Large shared libraries can force coordinated releases.

Be particularly cautious with shared:
- ORM entities.
- Repository layers.
- Domain models.
- Business rules owned by another service.

For example, sharing Payment Service’s internal entity classes couples consumers to its persistence design.

Prefer explicit API/event contracts, with backward-compatible evolution and contract tests.