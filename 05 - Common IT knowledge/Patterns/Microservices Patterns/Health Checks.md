**Purpose:** Tell infrastructure whether an instance should start receiving traffic, continue receiving traffic, or be restarted.

These are different questions:

| Probe     | Question                                                | Typical consequence of failure                   |
| --------- | ------------------------------------------------------- | ------------------------------------------------ |
| Startup   | Has initialization completed?                           | Allow startup time; restart if it never succeeds |
| Liveness  | Is this process stuck or irrecoverably unhealthy?       | Restart it                                       |
| Readiness | Can this instance currently serve its intended traffic? | Stop routing traffic to it                       |

Spring Boot Actuator provides health endpoints and supports liveness/readiness groups, commonly exposed as:

```http
/actuator/health/liveness
/actuator/health/readiness
```

### Important limitation: do not create restart cascades

If Payment Service is down, restarting every Order Service instance usually achieves nothing.

Therefore:

- Liveness should generally not depend on remote services.
- Readiness should include dependencies only when their failure truly prevents useful service.

For example, Order Service may still accept a pending order while payment is unavailable.

This connects to **[[Circuit Breaker]]s** and **graceful degradation**.

Also, a successful health check does not prove business correctness. A healthy process can still have a growing outbox backlog or stuck Sagas; those require metrics and alerts.