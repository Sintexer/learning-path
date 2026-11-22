---
flashcards:
  q-ym6s: { nid: 1795356717198, hash: mrzgr8fq, sync: qekifwv4 }
  q-ay3q: { nid: 1795356717217, hash: 26kns538, sync: 29rfpaav }
  q-ztbs: { nid: 1795356717241, hash: npmsnzk4, sync: 9jdi3mzb }
---

## Best Practices

1. **Establish Clear Guidelines** — Form project-wide code style rules and a review checklist for common issues to look for.
2. **Automate Where Possible** — Use static analysis tools, formatters, linters, and CI/CD checks so humans never spend time debating formatting.
3. **Focus on the Code, Not the Author** — Provide constructive feedback and avoid personal criticism ("this query does a full table scan" instead of "you wrote a slow query").
4. **Review Small, Incremental Changes** — Encourage small pull requests (~200–400 lines) and perform frequent, timely reviews (target < 24h turnaround) so PRs do not block team flow.
5. **Prioritize Important Issues** — Focus on correctness, security, concurrency, and operability first. Label minor items clearly (e.g., **Conventional Comments**: `nit:`, `suggestion:`, `question:`, `blocking:`) and do not let nits block approval.
6. **Document and Share Knowledge** — Encourage concise comments explaining *why* (trade-offs, invariants), not *what* the code already states.
7. **Use a Standard Review Process** — Establish a consistent workflow for submitting, reviewing, and merging code changes.
8. **Follow Up on Feedback** — Ensure all pull request comments are acknowledged by the change's author before merge.

## Staff-Level Review Lens

Beyond syntax and unit tests, high-leverage reviews check production operability:
- **Failure modes & resilience**: Timeouts ([[Timeout Pattern]]), [[Idempotency]], [[Retry Pattern|retries]], partial failure handling.
- **Deployment safety**: [[Backward Compatibility]], expand-and-contract [[Schema Migration]], and [[Zero-downtime deployment]].
- **Observability & security**: Metrics/traces ([[OpenTelemetry]]), [[Authorization]] / object-level ownership checks, no leaked PII in logs.

## Common Pitfalls

1. **Inconsistent Standards**
2. **Overly Critical or Personal Feedback**
3. **Large, Unmanageable Pull Requests**
4. **Ignoring Automation**
5. **Lack of Focus on Critical Issues**
6. **Reviewer Fatigue**
7. **Lack of Collaboration**
8. **Ignoring Feedback**

> [!warning] Failure modes & trade-offs
> - **The 2,000-line "LGTM" trap**: Oversized PRs trigger reviewer fatigue—reviewers rubber-stamp them after skimming, letting subtle concurrency or data-loss bugs slip into production. Split large changes into stacked PRs or behind feature flags.
> - **Bikeshedding**: Spending 15 comments debating variable names while missing an unindexed query, missing [[Authorization]] check, or non-idempotent retry. Automate style and explicitly distinguish `blocking:` from `nit:`.

Always separate ==blocking correctness/security/operability defects== from ==non-blocking `nit:` style suggestions== so reviews protect quality without stalling delivery.
^q-ym6s

Why should style and formatting rules be enforced in CI/CD rather than by human code reviewers?::Automating style removes subjective bikeshedding, eliminates reviewer fatigue, and focuses human review bandwidth on architecture, correctness, concurrency, and failure modes.
^q-ay3q

## What should a Staff-level code review verify beyond basic logic and unit tests? #card

1. **Failure modes & resilience**: Timeouts, idempotency, bounded retries, and graceful degradation.
2. **Deployment & data safety**: [[Backward Compatibility]], zero-downtime [[Schema Migration]], and rollback safety.
3. **Security & observability**: Object-level [[Authorization]], input validation, structured logs without PII, and metrics/tracing ([[OpenTelemetry]]).
^q-ztbs

## Related

- [[Team management]]
- [[Backward Compatibility]]
- [[Schema Migration]]
- [[Zero-downtime deployment]]
- [[OpenTelemetry]]
- [[Authorization]]
