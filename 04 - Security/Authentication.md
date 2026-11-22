---
flashcards:
  q-c5qi: { nid: 1795256794775, hash: b3d3ehr3, sync: vkyqssgn }
  q-hyc9: { nid: 1795356732775, hash: 9fw6ek92, sync: usd2nnjy }
  q-tcre: { nid: 1795356732798, hash: 5xc32ugr, sync: w4mf7sk9 }
  q-v9va: { nid: 1795356732818, hash: 3uttszjq, sync: 98cc66nt }
---

Usually users uses the combination of an email address and a password to verify your identity. If a user types in the correct email address and password, the system assumes the user is allowed to enter and grants them access. This is the process of **authentication**.  
  
==**Authentication** ensures that the user is who they say they are.== User names and passwords are the most common types of authentication. But you might also work with other forms, such as token-based authentication or biometric data, like a fingerprint. Authentication simply answers the question, “Are you who you say you are?”
^q-c5qi

## Authentication Factors & Modern Patterns

- **Three factors**: Something you *know* (password, PIN), something you *have* (hardware security key, authenticator app, passkey), something you *are* (biometrics). Combining two or more gives **MFA**.
- **Stateful vs Stateless**:
  - **Stateful sessions**: Server issues an opaque session ID stored in [[Redis]]/DB and sent via `HttpOnly; Secure; SameSite` cookies. Easy instant revocation, requires shared session store on the hot path.
  - **Stateless tokens**: [[Identity provider|IdP]] issues signed [[JWT]]s via [[OIDC]]. Scales horizontally without DB lookups per request, but immediate revocation requires short TTLs + refresh token rotation or a denylist.
- **Service-to-Service**: In [[Microservices architecture|microservices]] and [[Zero-Trust Security]], prefer mTLS (via [[Service Mesh]]) or short-lived workload identity tokens over shared static API keys.

> [!warning] Failure modes & trade-offs
> - **Password hashing**: Never hash passwords with fast hashes (SHA-256/MD5). Always use memory-hard, salted algorithms (**Argon2id**, scrypt, or bcrypt) to slow offline brute-force attacks.
> - **Credential stuffing & brute force**: Protect login endpoints with [[Rate Limiting]], progressive backoff, and breached-password checks.
> - **Session fixation**: Always rotate the session identifier immediately after login or privilege change.

Why must passwords be hashed with Argon2id or bcrypt instead of SHA-256?::SHA-256 is designed to be fast (billions of guesses/sec on GPUs), whereas Argon2id/bcrypt are intentionally slow, salted, and memory-hard to resist offline brute-force attacks.
^q-hyc9

## Stateful sessions vs stateless JWTs: what is the core trade-off? #card

- **Stateful sessions (cookies + Redis/DB)**: Instant revocation on logout/compromise and minimal payload size, at the cost of a shared state lookup on every request.
- **Stateless JWTs**: Zero DB lookup to verify signature locally (great for [[API Gateway]] and cross-service fan-out), at the cost of harder immediate revocation before `exp` and larger request headers.
^q-tcre

Always rotate session IDs on login and privilege changes to prevent session fixation attacks. #card-reminder
^q-v9va

## Related

- [[Authentication and Authorization]]
- [[Authorization]]
- [[JWT]]
- [[OIDC]]
- [[SAML]]
- [[Identity provider]]
- [[Identity federation]]
- [[Zero-Trust Security]]
- [[Rate Limiting]]