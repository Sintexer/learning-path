---
flashcards:
  q-uyrz: { nid: 1795356728992, hash: 7zngfc4m, sync: 8xgidr9b }
  q-2g8d: { nid: 1795356729017, hash: ch6wt3iu, sync: bhknke69 }
---

A [[Database]] provisioned, operated, and scaled on a cloud platform — either self-managed on cloud VMs ([[IaaS]]) or delivered as a fully managed **Database-as-a-Service (DBaaS)** (e.g., Amazon RDS/Aurora, DynamoDB, Google Cloud Spanner, Azure Cosmos DB, Snowflake).

## Deployment Models

- **Managed traditional [[RDBMS]] (e.g., AWS RDS, Cloud SQL):** Automates OS patching, minor upgrades, automated backups, point-in-time recovery, and Multi-AZ [[Leader-Based Replication|standby failover]], while keeping the familiar [[PostgreSQL]]/MySQL engine.
- **Cloud-native disaggregated storage (e.g., AWS Aurora, Neon):** Decouples stateless compute nodes from a distributed, multi-AZ shared storage layer — enabling fast read-replica scaling, instant branching, and rapid crash recovery without replaying full disks.
- **Globally distributed / NewSQL & Serverless (e.g., Cloud Spanner, CockroachDB, DynamoDB):** Built for [[Horizontal scaling]], multi-region [[Availability]], and elastic [[Serverless]] capacity using consensus protocols ([[Raft]]/[[Paxos]]) or [[Data partitioning|automatic sharding]].

> [!warning] Failure modes & trade-offs
> - **Managed != zero ops:** A cloud provider handles hardware and failover, *not* bad [[Database Index|indexes]], connection storms, [[PostgreSQL#VACUUM|autovacuum bloat]], or locking [[Schema Migration|migrations]].
> - **Hidden egress & IOPS costs:** Cross-AZ replication traffic, provisioned IOPS, backup snapshots, and nat-gateway/egress fees can dominate the monthly bill at scale.
> - **Vendor lock-in vs. velocity:** Cloud-native engines (Aurora, Spanner, DynamoDB) simplify scaling and multi-region [[Active-active systems|active-active]] deployments, but make migrating off the provider expensive.
> - **Failover blast radius:** Even managed Multi-AZ failover takes 30–60 seconds of DNS propagation + connection pool reset — applications must handle transient write failures with [[Retry Pattern|retries]] and [[Idempotency]].

## Flashcards

How do cloud-native databases like AWS Aurora differ architecturally from traditional managed RDS instances?::They decouple stateless compute instances from a shared, distributed, multi-AZ storage layer so read replicas share storage without duplicating disk writes.
^q-uyrz

What operational responsibilities still belong to the engineering team even when using a fully managed DBaaS?::Schema design, indexing, query optimization, connection pooling, safe [[Schema Migration|migrations]], and tuning engine-specific maintenance (such as [[PostgreSQL]] `autovacuum`).
^q-2g8d

## Related

- [[Database]]
- [[DBMS]]
- [[RDBMS]]
- [[NoSQL database]]
- [[Cloud-Native App]]
- [[Serverless]]
- [[Availability]]
- [[Horizontal scaling]]
- [[Active-active systems]]
- [[Active-passive systems]]
