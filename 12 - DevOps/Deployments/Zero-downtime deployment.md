---
flashcards:
  q-avz9: { nid: 1795356716117, hash: 346ijvvz, sync: zr7pu3ba }
  q-zkjh: { nid: 1795356716143, hash: uppt6hdg, sync: irxbe69i }
  q-mmav: { nid: 1795356716167, hash: q78wicmv, sync: 7mki2c6n }
  q-vguh: { nid: 1795356716192, hash: hxg2jkuh, sync: mak73v9x }
---

**Zero-downtime deployment** ensures an application does not lose [[Availability|availability]] or drop in-flight requests while rolling out a new version.

## Deployment Strategies

### Rolling Update

> Default in [[Kubernetes]] ([[K8S Deployment]])

Incrementally replaces instances of the old version (`v1`) with the new version (`v2`), controlled by `maxUnavailable` and `maxSurge`. Resource-efficient, **but `v1` and `v2` run simultaneously**, so APIs, serialized messages, caches, and database schemas must remain [[Backward Compatibility|backward]] and [[Forward Compatibility|forward compatible]].

### Blue-Green Deployment

You run two identical environments: **Blue** (current live) and **Green** (new target). You deploy `v2` to Green, run smoke tests, then flip the [[Load Balancing|load balancer]] or [[DNS]] router to Green. Gives instant cutover and instant rollback, but temporarily doubles compute cost and still requires DB compatibility if both share one database.

### Canary Deployment

Routes a small percentage of live traffic (e.g., 1% -> 5% -> 25% -> 100%) to `v2` via an L7 [[Reverse proxy]], [[API Gateway]], or [[Service Mesh]]. Automated analysis compares error rates, latency, and business metrics against `v1` and rolls back automatically on regression, minimizing blast radius. (For risky changes without user impact, pair with [[Shadow Testing]].)

## Zero-Downtime Database Migrations (Expand-and-Contract)

Stateless compute deployment is the easy part. **[[Schema Migration|Database schema updates]] are where zero-downtime usually breaks.**

> [!example]
> In a Rolling Update, `v1` code and `v2` code run at the *exact same time*. If `v2` executes a migration that drops or renames a column, `v1` pods instantly crash with `SQLGrammarException`!

Use the **Expand-and-Contract (Parallel Change)** pattern across multiple releases:
1. **Expand**: add the new column/table (nullable or with default), deploy code that **dual-writes** old and new fields while still reading the old one.
2. **Backfill**: migrate existing historical rows in small throttled batches (avoiding long table locks).
3. **Switch reads**: deploy code that reads from the new column (still dual-writing for safe rollback).
4. **Contract**: once no `v1` instances remain, stop writing the old column and drop it in a final release.

> [!warning] Failure modes & trade-offs
> - **Metadata / table locks during DDL**: in [[PostgreSQL]], `ALTER TABLE ... ADD COLUMN` with a volatile default or `CREATE INDEX` without `CONCURRENTLY` acquires `ACCESS EXCLUSIVE` / write locks that queue all live queries. Always set `lock_timeout` and use `CREATE INDEX CONCURRENTLY`.
> - **Mixed-version serialization**: if `v2` writes a new JSON/Protobuf/[[Redis]] cache format that `v1` cannot deserialize, rolling updates fail during overlap (see [[Backward Compatibility]] and [[Forward Compatibility]]).
> - **Missing graceful drain**: without readiness [[Health Checks]] and `preStop` draining (see [[JVM App Deployment]]), rolling updates drop active connections.

## Flashcards

What are the three main zero-downtime deployment strategies and their key trade-offs?::**Rolling** (resource-efficient, mixed `v1`/`v2` overlap), **Blue-Green** (instant cutover/rollback, 2x infra cost), and **Canary** (smallest blast radius via progressive traffic shifting, needs L7 routing + metrics).
^q-avz9

To evolve a database schema without downtime while `v1` and `v2` run simultaneously, use the ==Expand-and-Contract== pattern (add new column + dual-write -> backfill -> switch reads -> drop old column in a later release).
^q-zkjh

Never deploy a destructive schema change (drop/rename column) in the same release that removes code referencing it. #card-reminder
^q-mmav

### Why can a seemingly harmless DDL migration still cause a production outage in PostgreSQL? #card

DDL statements like `ALTER TABLE` or non-concurrent `CREATE INDEX` can acquire heavy table locks (`ACCESS EXCLUSIVE` / `SHARE`), blocking reads and writes while waiting behind a long-running query—creating a lock queue that stalls the entire service. Mitigate with `lock_timeout` and `CREATE INDEX CONCURRENTLY`.
^q-vguh

## Related
- [[JVM App Deployment]]
- [[Schema Migration]]
- [[K8S Deployment]]
- [[Backward Compatibility]]
- [[Forward Compatibility]]
- [[Shadow Testing]]
- [[Health Checks]]
- [[Availability]]
