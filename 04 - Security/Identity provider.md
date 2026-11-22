---
flashcards:
  q-s9mv: { nid: 1795356732541, hash: bvmd393f, sync: hepybpkf }
---

An **identity provider ([[IdP]])** is **a system that creates, stores, and manages digital identities**. The IdP can either directly authenticate the user ([[Authentication]]) or provide authentication services to third-party service providers (apps, websites, or other digital services) via [[Identity federation]]. Simply put, an IdP offers user authentication as-a-service (examples: [[Keycloak]], Okta, Microsoft Entra ID, Auth0).

## Core Responsibilities

- **Identity lifecycle**: User creation, MFA enrollment, password/passkey verification, and automated offboarding (SCIM).
- **Protocol & token issuance**: Issues signed [[JWT]]s via [[OIDC]] or XML assertions via [[SAML]], plus OAuth 2.0 access tokens for [[Authorization]].
- **Key management**: Rotates asymmetric signing keys and publishes active public keys via JWKS (`/.well-known/jwks.json`) or SAML metadata so services verify tokens locally.

> [!warning] Failure modes & trade-offs
> - **Tier-0 Single Point of Failure**: If the IdP is unreachable, all new logins and token refreshes fail across every dependent app. Mitigate with multi-region HA and local JWKS caching at the [[API Gateway]] so in-flight tokens keep validating.
> - **JWKS cache stampede**: If an attacker sends random unknown `kid` values in forged [[JWT]]s and every service immediately re-fetches JWKS from the IdP, it can self-DDoS the IdP. Always rate-limit / coalesce JWKS refreshes.
> - **Offboarding lag**: Disabling a user in the IdP does not revoke already-issued stateless tokens until they expire unless paired with short TTLs, refresh token revocation, or back-channel logout.

## What is the JWKS cache stampede failure mode, and how do you prevent it? #card

If verifiers blindly re-fetch the IdP's JWKS endpoint whenever they see an unknown `kid` header in a [[JWT]], an attacker can flood services with random `kid` values and trigger a self-inflicted DDoS against the IdP.

**Prevention**: Coalesce concurrent JWKS fetches (singleflight) and enforce a minimum cooldown interval between refreshes.

Why does disabling a user in the IdP not immediately terminate their access to stateless microservices?::Already-issued stateless JWTs remain valid until their `exp` timestamp unless services also check a revocation denylist or wait for short token TTLs to expire.
^q-s9mv

## Related

- [[IdP]]
- [[Identity federation]]
- [[Keycloak]]
- [[OIDC]]
- [[SAML]]
- [[JWT]]
- [[Authentication]]
- [[API Gateway]]
