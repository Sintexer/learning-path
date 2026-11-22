A cloud-native application is one that is designed from the ground up to exploit cloud platforms for auto-[[Scaling|scaling]], self-healing, rolling updates, rollbacks, and resilient operation under constant infrastructure churn.

## Core Characteristics

- **Packaged as [[Container|containers]]**: immutable artifacts built once and promoted across environments, typically adhering to [[Open Container Initiative]] standards and orchestrated by [[Kubernetes]].
- **Stateless compute + externalized state**: application instances are ephemeral ("cattle, not pets"); durable state lives in managed databases, object storage, or caches like [[Redis]].
- **Loosely coupled [[Microservices App|microservices]]**: bounded services communicating over [[HTTP]], [[gRPC]], or asynchronous events ([[Event-driven architecture]]).
- **Declarative infrastructure & delivery**: environments are provisioned via [[IaaC]] (e.g., [[Terraform]]) and deployed with [[Zero-downtime deployment]] strategies.
- **Built-in observability & resilience**: exposes [[Health Checks]] (liveness/readiness), structured logs, metrics, and [[Distributed Tracing]] ([[OpenTelemetry]]), and handles partial failures with [[Timeout Pattern|timeouts]], [[Retry Pattern|retries]], and [[Circuit Breaker|circuit breakers]].
- **Externalized configuration**: config is injected via environment variables or an [[External Configuration Store]] rather than baked into images (Twelve-Factor App).

> [!warning] Failure modes & trade-offs
> - **Lift-and-shift trap**: putting a stateful monolith into a [[Container]] does not make it cloud-native—if it stores files on local disk, takes 3 minutes to boot, or ignores `SIGTERM`, autoscaling and rolling updates will cause outages.
> - **Operational complexity**: cloud-native architecture trades hardware management for distributed-systems complexity (networking, [[Service discovery]], [[Service Mesh]], eventual consistency).

## Flashcards

Cloud-native application::An application designed around ephemeral [[Container|containers]], externalized state, declarative [[IaaC]], automated [[Scaling|scaling]], self-healing, and built-in observability.

Why is "lift-and-shift into containers" not enough to make an app cloud-native?::Containers alone do not fix local disk state, slow startup, missing [[Health Checks]], or lack of graceful shutdown—all of which break autoscaling and rolling updates.

Design cloud-native instances as disposable cattle: assume any instance can be killed and replaced at any moment without warning. #card-reminder

### What core traits enable self-healing and auto-scaling in a cloud-native app? #card

- Stateless, fast-starting instances with externalized config and state.
- Accurate readiness/liveness [[Health Checks]] and graceful `SIGTERM` handling.
- Idempotent operations ([[API Idempotency]]) so restarts and retries are safe.

## Related
- [[Microservices App]]
- [[Container]]
- [[Kubernetes]]
- [[IaaC]]
- [[Zero-downtime deployment]]
- [[Health Checks]]
- [[OpenTelemetry]]
