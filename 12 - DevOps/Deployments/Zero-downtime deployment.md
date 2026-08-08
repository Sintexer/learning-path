
Zero-downtime deployment ensures app does not loos availability during update.

## Deployment strategies

### Rolling update

> Default in [[Kubernetes]]

This process incrementally replaces instances of the old version (v1) with the new version (v2), one pod at a time. It is resource-efficient. But backend must be able to handle mixed traffic for the time of update.

### Green-Blue deployment

You run two identical physical environments: **Blue** (Current Live), **Green** (Idle New Target). You deploy v2 to Green, test it fully, then flip the Load Balancer / DNS router to point to Green. Instant update, instant rollback. But it costly: for some tome it is required to run twice as much instances.

### Canary deployment

Routes a tiny percentage of live user traffic (e.g., 1% or 5%) to `v2`, while 95% stays on `v1`. Automated systems monitor error rates, latency (Prometheus/Datadog), and if health metrics pass, gradually scale `v2` up to 100%. Minimizes blast radius of bad code updates.

## Zero-Downtime Database migration

nfrastructure deployment is simple. **Database schema updates are where zero-downtime usually breaks.**

> [!example]
> In a Rolling Update, `v1` code and `v2` code run at the _exact same time_. If `v2` executes a schema migration that drops a column, `v1` code instantly crashes with `SQLGrammarException`!


