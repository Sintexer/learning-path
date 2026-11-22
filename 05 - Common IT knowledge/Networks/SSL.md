---
flashcards:
  q-uvfm: { nid: 1795356723292, hash: aaa3zeig, sync: pr9bx7kg }
  q-54vw: { nid: 1795356723317, hash: cguj9hja, sync: tx5n6hen }
  q-w55r: { nid: 1795356723342, hash: 3znh3qb9, sync: 7e6y838w }
---

> AKA **Secure Sockets Layer** (deprecated predecessor to [[TLS]])

**SSL** was the original cryptographic protocol developed by Netscape in the mid-1990s to encrypt network traffic between clients and servers (giving rise to [[HTTPS]]). Although people still colloquially say "SSL certificate", **all versions of SSL are deprecated and insecure**—modern systems exclusively use [[TLS]] (1.2 and 1.3).

## Version History & Why SSL Was Retired

- **SSL 1.0**: never publicly released (serious security flaws).
- **SSL 2.0 (1995)**: deprecated in 2011 (RFC 6176) — weak MACs, vulnerable to downgrade and truncation attacks.
- **SSL 3.0 (1996)**: deprecated in 2015 (RFC 7568) — broken by the **POODLE** padding-oracle attack against CBC ciphers.
- **[[TLS]] 1.0 / 1.1 (1999 / 2006)**: deprecated in 2021 (RFC 8996) — vulnerable to BEAST/weak ciphers.
- **[[TLS]] 1.2 (2008) & TLS 1.3 (2018)**: current secure standards (AEAD ciphers like [[AES256|AES-GCM]] and ChaCha20-Poly1305, mandatory Perfect Forward Secrecy in TLS 1.3).

> [!warning] Failure modes & trade-offs
> - **Downgrade attacks**: leaving SSL 3.0 or TLS 1.0/1.1 enabled on a [[Reverse proxy]] or load balancer for "legacy clients" lets a MITM downgrade connections to broken ciphers. Disable everything below TLS 1.2 (prefer TLS 1.3).

## Flashcards

SSL:::Secure Sockets Layer — the deprecated predecessor to [[TLS]]; all SSL versions (1.0–3.0) are cryptographically broken and must be disabled.
^q-uvfm

Which attack forced the final deprecation of SSL 3.0?::POODLE (Padding Oracle On Downgraded Legacy Encryption) against CBC-mode ciphers.
^q-54vw

Modern production endpoints should disable SSL and TLS 1.0/1.1, allowing only ==[[TLS]] 1.2== (with AEAD/PFS cipher suites) and ==TLS 1.3==.
^q-w55r

## Related
- [[TLS]]
- [[HTTPS]]
- [[AES256]]
- [[Reverse proxy]]
