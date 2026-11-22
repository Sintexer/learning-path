> AKA **Hypertext Transfer Protocol Secure**

**HTTPS** is [[HTTP]] layered over [[TLS]] (historically [[SSL]]), using default port **443** ([[TCP]] for [[HTTP1.1]]/[[HTTP2]], [[UDP]] for [[HTTP3]]/QUIC).

## What HTTPS Guarantees (and What It Doesn't)

- **Confidentiality**: encrypts headers, URL path/query parameters, cookies, and request/response bodies so network intermediaries cannot read them.
- **Integrity**: AEAD ciphers detect any in-transit tampering or injection.
- **Server [[Authentication]]**: X.509 certificates signed by a trusted CA prove the client is talking to the real domain (and optional **mTLS** authenticates the client too).
- **Still visible to network observers**: destination IP/port, [[DNS]] lookup (unless DoH/DoT), and hostname via SNI (unless Encrypted Client Hello / ECH is used), plus packet sizes and timing.

## Production Hardening

- **HSTS (`Strict-Transport-Security`)**: instructs browsers to refuse plaintext HTTP for the domain (prevents SSL-stripping downgrade attacks); add `includeSubDomains; preload` once all subdomains support HTTPS.
- **TLS Termination vs End-to-End TLS**: edge [[Reverse proxy|reverse proxies]] / [[Load Balancing|load balancers]] often terminate public HTTPS, but sensitive or [[Zero-Trust Security]] environments re-encrypt internal hops (mTLS via [[Service Mesh]]).
- **Automated Certificate Rotation**: use ACME (Let's Encrypt) or `cert-manager` in [[Kubernetes]] so certificates never expire manually.

> [!warning] Failure modes & trade-offs
> - **Mixed content & redirect loops**: if a [[Reverse proxy]] terminates HTTPS and forwards plaintext HTTP to the backend without `X-Forwarded-Proto: https`, apps that enforce HTTPS enter an infinite `301` redirect loop.
> - **HSTS lock-in**: enabling `Strict-Transport-Security: max-age=31536000; includeSubDomains` breaks any internal or legacy subdomain that lacks a valid TLS cert until `max-age` expires.

## Flashcards

HTTPS:::[[HTTP]] over [[TLS]] (default port 443), providing confidentiality, integrity, and CA-backed server authentication.

What parts of an HTTPS request remain visible to a network eavesdropper (without ECH/DoH)?::Destination IP/port, [[DNS]] query, TLS SNI hostname, and traffic size/timing (while URL path, query params, headers, and body are encrypted).

What HTTP header protects against SSL-stripping downgrade attacks by forcing browsers to use HTTPS?::`Strict-Transport-Security` (HSTS).

## Related
- [[HTTP]]
- [[TLS]]
- [[SSL]]
- [[Reverse proxy]]
- [[Zero-Trust Security]]
- [[Service Mesh]]
