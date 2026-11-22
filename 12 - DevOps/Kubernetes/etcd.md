**etcd** is a strongly consistent ([[Linearizability|linearizable]]), distributed key-value store used as the single source of truth for [[Kubernetes]] cluster state and configuration.

At its core, `etcd` uses the **[[Raft]]** consensus algorithm to replicate a write-ahead log ([[WAL]]) across a cluster of odd-numbered nodes, ensuring strong consistency and fault tolerance across node failures and network partitions (CP in the [[CAP Theorem]]).

In [[Kubernetes]], **only the [[K8S Control Plane#API Server|API Server]] talks to `etcd`**. Every Pod, Deployment, Service, Secret, Lease, and ConfigMap is persisted in `etcd` with a monotonically increasing **revision** (enabling `kube-apiserver` **watch** streams and [[Optimistic Concurrency Control]] via `resourceVersion`).

## Quorum Math & Cluster Sizing

`etcd` requires a strict majority (**[[Quorum Consensus|quorum]]**) of \(Q = \lfloor N/2 \rfloor + 1\) members to elect a leader and commit writes:

| Cluster Size (\(N\)) | Quorum Needed (\(Q\)) | Fault Tolerance (\(N - Q\)) | Recommendation |
| --- | --- | --- | --- |
| 1 | 1 | 0 | Local dev only |
| **3** | 2 | 1 | Standard HA production (3 AZs) |
| 4 | 3 | 1 | Never use (same fault tolerance as 3, worse write latency) |
| **5** | 3 | 2 | High-resilience production (survives 1 maintenance + 1 failure) |
| 7+ | 4+ | 3+ | Rarely worth the Raft write-latency penalty |

> [!warning] Failure modes & trade-offs
> - **Quorum loss (freeze)**: if fewer than \(\lfloor N/2 \rfloor + 1\) nodes are reachable (e.g., 2 of 3 fail), `etcd` loses quorum and refuses all writes (preventing [[Split-Brain]]). Running Pods keep serving, but no new scheduling, failover, or secret updates can happen until quorum is restored or recovered from a periodic `etcdctl snapshot`.
> - **Disk `fsync` latency kills Raft**: `etcd` calls `fsync` on every Raft WAL write. Slow shared network disks or noisy neighbors spike `wal_fsync_duration_seconds` (>10–25ms), causing missed leader heartbeats and **leader election storms**. Always give `etcd` dedicated low-latency NVMe/SSD storage.
> - **DB size quota (`mvcc: database space exceeded`)**: `etcd` keeps MVCC history of all key revisions (default hard quota 2GB–8GB). Without periodic **compaction** and **defragmentation**, `etcd` triggers a `NOSPACE` alarm and flips the entire cluster read-only!
> - **Secrets at rest**: by default Kubernetes stores `Secret` objects in `etcd` base64-encoded (not encrypted)—always enable KMS Encryption-at-Rest for `etcd` in production.

## Flashcards

etcd:::Strongly consistent, [[Raft]]-backed distributed key-value store serving as the sole persistent state store for [[Kubernetes]].

Why should an `etcd` cluster always have an **odd** number of nodes (3 or 5) rather than 4?::A 4-node cluster requires quorum 3 (tolerating only ==1== failure)—the exact same fault tolerance as a ==3-node== cluster, while adding extra [[Raft]] replication overhead and split-vote risk.

What happens to a [[Kubernetes]] cluster if `etcd` loses [[Quorum Consensus|quorum]] (e.g., 2 of 3 members go down)?::`etcd` stops accepting writes to prevent ==[[Split-Brain]]==; already-running Pods keep serving traffic, but no new Pods, rollouts, EndpointSlice updates, or failovers can occur until quorum is restored.

### Why is dedicated low-latency SSD/NVMe storage mandatory for `etcd` in production? #card

Every [[Raft]] commit requires an `fsync` to the [[WAL]] before acknowledging the leader. Slow disk I/O causes `etcd` heartbeats to time out, triggering continuous leader election storms that destabilize the entire `kube-apiserver` and cluster.

## Related
- [[K8S Control Plane]]
- [[Kubernetes]]
- [[Raft]]
- [[Quorum Consensus]]
- [[Linearizability]]
- [[Split-Brain]]
- [[CAP Theorem]]
- [[WAL]]
- [[MVCC]]
