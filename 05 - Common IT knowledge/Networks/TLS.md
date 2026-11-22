---
flashcards:
  q-xzz2: { nid: 1795356722967, hash: 999uwyug, sync: qraa89wu }
  q-v8f5: { nid: 1795356722995, hash: k3ihxsyi, sync: xt2h32p4 }
  q-pd8p: { nid: 1795356723018, hash: xkvwp3fb, sync: 6wnj22df }
  q-qzhd: { nid: 1795356723045, hash: xk4369k9, sync: 6nm9kmzn }
  q-9hx4: { nid: 1795356723068, hash: uxyiccb3, sync: x6raufx2 }
---

**Transport Layer Security (TLS)** and its deprecated predecessor, [[SSL]], are cryptographic protocols designed to provide secure communication between clients and servers (powering [[HTTPS]], mTLS in [[Service Mesh|service meshes]], and encrypted DB connections) by ensuring **confidentiality**, **integrity**, and **authenticity**.

## Deep Dive into TLS

1. **Protocol Overview:**
   TLS operates on top of a reliable transport (usually [[TCP]], or integrated into QUIC over [[UDP]] for [[HTTP3]]), securing application protocols like [[HTTP]], [[gRPC]], SMTP, and database wire protocols (mapping conceptually near [[OSI 6 layer|OSI Layers 5–6]]).

2. **Handshake (TLS 1.2 vs TLS 1.3):**
   - **TLS 1.2 (2 RTTs)**:
     a. `ClientHello`: TLS version, supported cipher suites, `ClientRandom`, SNI/ALPN extensions.
     b. `ServerHello` + `Certificate` + `ServerKeyExchange` + (optional `CertificateRequest` for mTLS) + `ServerHelloDone`.
     c. `ClientKeyExchange` (pre-master secret / key share) + `ChangeCipherSpec` + `Finished`.
     d. Server `ChangeCipherSpec` + `Finished`.
   - **TLS 1.3 (1 RTT, or 0-RTT resumption)**:
     Drops weak static RSA key exchange and legacy ciphers; client guesses the key-exchange group (ECDHE) and sends its key share right in `ClientHello`, cutting handshake latency in half and encrypting certificates immediately.

3. **Encryption:**
   TLS uses **asymmetric cryptography** (ECDHE / RSA / ECDSA) during the handshake for authentication and key exchange, then switches to fast **symmetric AEAD encryption** (e.g., [[AES256|AES-256-GCM]] or ChaCha20-Poly1305) for bulk data transfer.

4. **Authentication & mTLS:**
   TLS authenticates the server via X.509 digital certificates signed by a trusted Certificate Authority (CA). In **mutual TLS (mTLS)**, the client also presents a certificate—standard for service-to-service identity in [[Zero-Trust Security]] and [[Service Mesh|service meshes]].

5. **Key Extensions:**
   - **SNI (Server Name Indication)**: sends the target hostname in plaintext during `ClientHello` so one IP/[[Reverse proxy]] can host multiple TLS certificates.
   - **ALPN (Application-Layer Protocol Negotiation)**: negotiates [[HTTP2]] vs [[HTTP1.1]] during the TLS handshake without an extra round trip.

6. **OCSP Stapling:**
   Enables the server to attach ("staple") a CA-signed timestamped validity proof for its certificate during the handshake, avoiding slow, privacy-leaking client OCSP lookups to the CA.

7. **Certificate Transparency (CT):**
   Requires CAs to publish all issued certificates in public, append-only logs so domain owners can detect misissued or unauthorized certificates.

8. **Perfect Forward Secrecy (PFS):**
   Ensures past recorded TLS sessions cannot be decrypted even if the server's long-term private key is later compromised. Achieved by ephemeral Diffie-Hellman (ECDHE) key exchange (mandatory in TLS 1.3).

9. **Certificate Pinning:**
   Client verifies the server certificate (or public key hash) against a hardcoded allowlist. Rarely recommended on the open web now because it causes self-inflicted outages during certificate rotation.

> [!warning] Failure modes & trade-offs
> - **Expired certificates**: one of the most common preventable outages—automate issuance and rotation (ACME / Let's Encrypt / cert-manager in [[Kubernetes]]) and alert 30 days before expiry.
> - **TLS 1.3 0-RTT replay risk**: 0-RTT early data is *not* replay-protected; only allow idempotent requests ([[API Idempotency]]) over 0-RTT, never state-mutating endpoints.
> - **Handshake latency on short-lived connections**: TCP (1 RTT) + TLS 1.3 (1 RTT) = 2 RTTs before the first byte; reuse connections via keep-alive / [[HTTP2]] / TLS session resumption.

## Flashcards

TLS:::Transport Layer Security — cryptographic protocol providing confidentiality, integrity, and authenticity over the network (successor to [[SSL]]).
^q-xzz2

How does TLS 1.3 cut handshake latency from 2 RTTs (TLS 1.2) to 1 RTT?::The client sends its ==ephemeral key share (ECDHE)== directly inside `ClientHello` instead of waiting for `ServerHello` first.
^q-pd8p
^q-v8f5

Perfect Forward Secrecy (PFS) guarantees that compromising a server's long-term private key cannot decrypt past traffic because session keys are derived via ==ephemeral Diffie-Hellman (ECDHE)==.
^q-qzhd

### Why is TLS 1.3 0-RTT early data unsafe for non-idempotent API endpoints? #card

Because 0-RTT early data can be captured and replayed by a network attacker before the server verifies freshness. Only safe, read-only, or strictly [[API Idempotency|idempotent]] requests should be accepted in 0-RTT.
^q-9hx4

## Related
- [[SSL]]
- [[HTTPS]]
- [[AES256]]
- [[TCP]]
- [[HTTP2]]
- [[HTTP3]]
- [[Reverse proxy]]
- [[Zero-Trust Security]]
- [[Service Mesh]]
