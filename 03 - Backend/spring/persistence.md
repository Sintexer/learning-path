---
flashcards:
  q-qzq2: { nid: 1795356733825, hash: dkxn6j3u, sync: it4yu8sk }
  q-t5ix: { nid: 1795356733845, hash: m6kyajq8, sync: vz66d63f }
  q-f24r: { nid: 1795356733868, hash: 4xc2r8vc, sync: qp5dfxfj }
---

# Spring Persistence

## Multi-Datasource Configuration

1. When we need to have 2 datasources in our application (2 DBs), we need to create two Spring configurations for datasources (`@Primary` and secondary), each with its own `DataSource`, `EntityManagerFactory`, and `PlatformTransactionManager`. Configure `@EnableJpaRepositories(basePackages = ..., entityManagerFactoryRef = ..., transactionManagerRef = ...)` and `@EntityScan` explicitly so Spring routes repositories to the right database.

## `@Transactional` Mechanics & Pitfalls

Spring implements `@Transactional` via **AOP proxies** (JDK dynamic proxy or CGLIB subclass) wrapping the bean:
- **Self-invocation bypass:** Calling `this.otherMethod()` inside the same bean bypasses the Spring proxy, so `@Transactional` on `otherMethod()` is **silently ignored**. Fix by extracting to a separate bean or using `TransactionTemplate`.
- **Rollback defaults:** By default, Spring rolls back only on **unchecked** exceptions (`RuntimeException`, `Error`), NOT checked exceptions—specify `rollbackFor = Exception.class` if checked exceptions must roll back.
- **Propagation:**
  - `REQUIRED` (default): joins the caller's [[Transaction|transaction]] or starts a new one.
  - `REQUIRES_NEW`: suspends the caller's transaction and opens a **second JDBC connection** for an independent transaction.
- **`readOnly = true`:** Hints [[Hibernate]] to skip dirty-checking snapshots and allows read-replica routing or DB-level read-only optimizations.

## Connection Pool & [[JPA]] Integration

- **HikariCP sizing:** Keep pools small and bounded (guided by [[Little's Law]] and DB CPU cores; see [[API Performance improvement]]).
- **Disable OSIV:** Set `spring.jpa.open-in-view=false` so JDBC connections are released as soon as `@Transactional` exits (see [[JPA]]).
- **Avoid external I/O inside `@Transactional`:** Never make HTTP/RPC calls or publish directly to [[Kafka]] while holding a DB transaction open—use the [[Outbox Pattern]] to avoid [[Dual-Write]] bugs and connection-pool starvation.

> [!warning] Failure modes & trade-offs
> - **`REQUIRES_NEW` connection-pool deadlock:** If a thread holds connection #1 in an outer transaction and calls a `REQUIRES_NEW` method that needs connection #2, a saturated pool deadlocks—every thread holds 1 connection and waits forever for a 2nd.
> - **Cross-datasource writes are NOT atomic:** Spring's `JpaTransactionManager` only covers a single datasource. Writing to 2 DBs in one request is a [[Dual-Write]] problem unless you use XA/[[Two-Phase Commit]] or an async pattern like [[Outbox Pattern]] / [[Saga]].
> - **`@Transactional` on `private` methods:** CGLIB proxies cannot intercept `private` methods, so `@Transactional` is silently ignored.

## Flashcards

## Why does calling a `@Transactional` method via `this.method()` inside the same Spring bean fail to start a transaction? #card
Spring applies `@Transactional` through an AOP proxy wrapping the bean. Internal self-invocation (`this.method()`) calls the target object directly, bypassing the proxy so the transaction interceptor never runs. Fix by moving the method to another bean or using `TransactionTemplate`.

By default, Spring's `@Transactional` rolls back automatically on ==unchecked exceptions (`RuntimeException` / `Error`)==, NOT on checked exceptions.
^q-qzq2

## How can `Propagation.REQUIRES_NEW` deadlock a Spring application's HikariCP connection pool? #card
`REQUIRES_NEW` suspends the outer transaction (keeping its JDBC connection checked out) and requests a second connection from the pool. If all pool connections are held by outer transactions waiting for a second connection, every thread blocks until `connectionTimeout`—causing a pool deadlock.
^q-t5ix

Never perform external HTTP calls or broker publishes inside a `@Transactional` method—release the DB connection quickly and use the [[Outbox Pattern]] for reliable event publishing. #card-reminder
^q-f24r

## Related
- [[JPA]]
- [[Hibernate]]
- [[Transaction]]
- [[Outbox Pattern]]
- [[Dual-Write]]
- [[API Performance improvement]]
- [[Little's Law]]
- [[Two-Phase Commit]]

