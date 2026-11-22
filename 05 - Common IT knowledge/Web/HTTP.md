---
flashcards:
  q-d25d: { nid: 1795356718567, hash: dfzix437, sync: dizitbgn }
  q-2tx6: { nid: 1795361687932, hash: bq7xgs3d, sync: r8iez54w }
  q-54dw: { nid: 1795356718641, hash: hpuxcuxv, sync: 4xxhm4za }
---

> AKA **Hypertext Transfer Protocol**

**HTTP** is a stateless request-response application protocol at [[OSI 7 layer]] (default port **80**; encrypted [[HTTPS]] uses port **443**).

There are three major versions in production:

- [[HTTP1.1]] (classic [[REST]], web)
- [[HTTP2]] (multiplexed binary framing, used by [[gRPC]] and modern web)
- [[HTTP3]] (QUIC over [[UDP]], eliminates [[TCP]] Head-of-Line blocking)

### Summary of Differences

| Feature | [[HTTP1.1\|HTTP/1.1]] | [[HTTP2\|HTTP/2]] | [[HTTP3\|HTTP/3]] |
| --- | --- | --- | --- |
| **Framing** | Text-based | Binary | Binary (over QUIC) |
| **Multiplexing** | No (1 in-flight per conn) | Yes (single TCP conn) | Yes (independent QUIC streams) |
| **Header Compression** | No | Yes (HPACK) | Yes (QPACK) |
| **Transport Protocol** | [[TCP]] | [[TCP]] | [[UDP]] (QUIC) |
| **Head-of-Line Blocking** | Yes (HTTP + TCP) | Yes (at [[TCP]] level) | No |
| **Connection Setup** | TCP + [[TLS]] (2–3 RTT) | TCP + [[TLS]] (2–3 RTT) | Combined QUIC + TLS 1.3 (1 RTT / 0-RTT) |
| **Encryption** | Optional ([[HTTPS]]) | Effectively required ([[TLS]]) | Mandatory (built into QUIC) |

### Method Semantics (Safety & [[Idempotency]])

- **Safe & Idempotent**: `GET`, `HEAD`, `OPTIONS` (read-only, cacheable).
- **Unsafe & Idempotent**: `PUT`, `DELETE` (repeating leaves the resource in the same state).
- **Unsafe & Non-Idempotent**: `POST`, `PATCH` (require an [[Idempotency Key]] / [[API Idempotency]] for safe retries).

> [!tip] Production reality
> HTTP/2 **Server Push** was deprecated and removed from Chrome/modern browsers due to cache-state ambiguity; **103 Early Hints** replaced it for preloading critical assets. Also remember that HTTP status codes drive automated infrastructure behavior: `429` / `503` with `Retry-After` signal [[Backpressure]] and [[Rate Limiting]], whereas retrying `4xx` client errors amplifies incidents.

## Flashcards

HTTP:::Hypertext Transfer Protocol — stateless request-response protocol at [[OSI 7 layer]] (port 80 plaintext, port 443 for [[HTTPS]]).
^q-d25d

Which HTTP methods are guaranteed to be idempotent by specification?::`GET`, `HEAD`, `OPTIONS`, `PUT`, and `DELETE` (`POST` and `PATCH` are ==not== idempotent by default).
^q-bhdk
^q-2tx6

HTTP/1.1 uses ==text framing over TCP==; HTTP/2 adds ==binary multiplexing and HPACK over TCP==; HTTP/3 moves to ==QUIC over UDP with QPACK== to eliminate TCP Head-of-Line blocking.
^q-54dw

## Related
- [[HTTPS]]
- [[HTTP1.1]]
- [[HTTP2]]
- [[HTTP3]]
- [[REST]]
- [[API Idempotency]]
- [[TCP]]
- [[UDP]]
