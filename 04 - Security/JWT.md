A **JSON Web Token (JWT)** is a compact, URL-safe, self-contained token format (RFC 7519) used to transmit signed claims between parties—most commonly for stateless [[Authentication]] and [[Authorization]] in [[OIDC]] and [[Microservices architecture|microservices]].

## Structure

A JWT consists of three Base64URL-encoded parts separated by dots (`header.payload.signature`):

1. **Header**: Metadata such as signing algorithm (`alg`: `HS256`, `RS256`, `ES256`), token type (`typ: "JWT"`), and key identifier (`kid`) used for key rotation.
2. **Payload (Claims)**: JSON object containing registered claims (`iss` issuer, `sub` subject, `aud` audience, `exp` expiration, `nbf` not before, `iat` issued at, `jti` unique token ID) and custom application claims (e.g., `roles`, `tenant_id`).
3. **Signature**: Cryptographic signature over `base64url(header) + "." + base64url(payload)`.
   - **Symmetric (`HS256`)**: Shared secret—every verifier can also mint tokens. Risky in multi-service systems.
   - **Asymmetric (`RS256`, `ES256`)**: The [[Identity provider|IdP]] (e.g., [[Keycloak]]) signs with a private key; downstream services and [[API Gateway]] instances verify locally using the public key fetched and cached from the JWKS endpoint (`/.well-known/jwks.json`).

A standard signed JWT (JWS) provides ==integrity and authenticity, not confidentiality==—anyone can Base64-decode the payload. Use JWE or omit sensitive data if confidentiality is required.

> [!warning] Failure modes & trade-offs
> - **Revocation problem**: Because verification is stateless, a compromised or logged-out JWT stays valid until `exp`. Mitigate with short access-token TTLs (5–15 minutes), refresh token rotation, and a fast denylist (e.g., [[Redis]] keyed by `jti`) for high-risk revocations.
> - **Algorithm confusion & `alg: none`**: Never trust the token header's `alg` blindly. An attacker can send `alg: none` or switch `RS256` to `HS256` and sign with the server's public key as the HMAC secret. Always pin allowed algorithms on the verifier.
> - **Clock skew**: Distributed servers drift; allow a small bounded leeway (30–60s) when checking `exp` and `nbf`.
> - **Token bloat**: JWTs travel on every [[HTTP]] request header. Stuffing hundreds of fine-grained permissions into claims bloats bandwidth and hits proxy header size limits (commonly 4–8 KB).

Keep JWT access tokens short-lived (5–15 min) and use rotating refresh tokens with reuse detection rather than trying to make long-lived JWTs revocable. #card-reminder

## How do you handle immediate logout or revocation with stateless JWTs? #card

Stateless JWTs cannot be invalidated on their own before `exp`. In production, combine:
1. **Short access-token TTL** (5–15 min) so exposure windows stay small.
2. **Refresh token rotation with reuse detection** stored server-side at the [[Identity provider|IdP]]—revoking the refresh token prevents new access tokens, and reuse of an old refresh token invalidates the whole token family.
3. **Emergency denylist** (e.g., [[Redis]] storing revoked `jti` or `user_id + logout_timestamp` with TTL equal to max access-token lifetime) checked at the [[API Gateway]] when immediate revocation is required.

## What is the JWT algorithm confusion attack (RS256 to HS256) and how do you prevent it? #card

If a verifier trusts the `alg` header in an incoming JWT, an attacker can change `alg` from `RS256` (asymmetric) to `HS256` (symmetric) and sign the forged token using the server's **public** RSA key as the HMAC shared secret.

**Prevention**: Pin the expected algorithm(s) explicitly in the verification library config, bind each `kid` to its allowed algorithm, and reject `alg: none`.

## Related

- [[OIDC]]
- [[Authentication]]
- [[Authorization]]
- [[Identity provider]]
- [[Keycloak]]
- [[API Gateway]]
- [[HTTPS]]
- [[Zero-Trust Security]]
