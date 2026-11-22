---
flashcards:
  q-6mkm: { nid: 1795356731897, hash: qm78mkpe, sync: a2sxqxc8 }
  q-ifh8: { nid: 1795356731917, hash: rbjkyd5b, sync: 9e4he573 }
  q-6p3j: { nid: 1795356731942, hash: bzmpgbpn, sync: 3bk5xzah }
---

> AKA **Server Side Encryption**

**Server-Side Encryption (SSE)** encrypts data **at rest** on the storage service immediately before writing to disk and transparently decrypts it on authorized reads—typically using [[AES256]] (GCM for objects, XTS for block volumes).

## Key Management Models

Using AWS S3 terminology (mirrored across GCP/Azure):

- **SSE-S3 (Provider-managed keys)**: Storage service manages and rotates keys automatically. Zero operational overhead, but no custom key policies, revocation control, or per-decrypt audit logs.
- **SSE-KMS (Customer-managed KMS keys)**: Uses envelope encryption (`DEK` wrapped by a KMS `KEK`). Provides fine-grained IAM/key policies ([[Least privilege principle]]), CloudTrail audit logs for every decrypt, and customer-controlled rotation/revocation.
- **SSE-C (Customer-provided keys)**: Client sends the encryption key on every [[HTTPS]] request; the cloud provider never stores the key. Highest control, highest operational risk (losing the key means permanent data loss).

When using SSE-KMS on high-throughput object storage, enable ==Bucket Keys (or DEK caching)== to avoid KMS API throttling and cost spikes.
^q-6mkm

> [!warning] Failure modes & trade-offs
> - **SSE does NOT protect against compromised app credentials**: Because decryption is transparent to any caller with storage read + `kms:Decrypt` permission, SSE defends against physical disk theft or unauthorized storage-layer access—not an attacker who hijacks an over-privileged app IAM role. Combine SSE with [[Least privilege principle]] and [[Zero-Trust Security]], or use **Client-Side Encryption (CSE)** when the storage provider itself must never see plaintext.
> - **KMS throttling under load**: Calling KMS per object on high-QPS workloads can hit KMS [[Rate Limiting|rate limits]], causing sudden 503 errors unless Bucket Keys or local DEK caching are enabled.

Why does Server-Side Encryption (SSE) not protect against a compromised application IAM role?::Because SSE decrypts transparently on authorized reads; any caller with both storage read and key decrypt permissions receives plaintext.
^q-ifh8

## Compare SSE-S3, SSE-KMS, and SSE-C key management models. #card

- **SSE-S3**: Provider owns and rotates keys; simplest, lowest cost, no custom audit trail or revocation.
- **SSE-KMS**: Customer controls KMS key policies, rotation, revocation, and per-decrypt audit logs; incurs KMS latency/cost/quotas (mitigated with Bucket Keys).
- **SSE-C**: Customer supplies the key on every request; provider never persists the key, but key loss equals permanent data loss.
^q-6p3j

## Related

- [[AES256]]
- [[TLS]]
- [[Least privilege principle]]
- [[Zero-Trust Security]]
- [[Rate Limiting]]
