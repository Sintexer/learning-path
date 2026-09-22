> **Central question:** How do we replace a production system without rewriting and switching everything at once?

The **Strangler Fig pattern** gradually replaces an existing system by moving selected capabilities to a new implementation.

During migration, both systems coexist:

```
			 Routing layer
				  |
	  ┌───────────┴───────────┐
	  v                       v
Legacy system             New services
Remaining features        Migrated features
```

Over time, the new system takes over more responsibilities until the old system can be retired. The pattern does not require microservices. You can use it to replace a legacy application with a new modular monolith.

It might sound tempting to migrate all at once - develop a fully new service and migrate. But this rarely succeeds. Many features might be undocumented, hidden integration, etc. These bugs will accumulate and fail all at once. The Strangler fig solves this by migrating one feature at a time, observing real prod behavior, fixing gaps, and moving further only after green signal.

Other patterns, like [[Shadow Testing]], [[Anti-Corruption Layer]] simplify the integration process.