---
flashcards:
  q-xspc: { nid: 1795356732642, hash: yim2sp3e, sync: mtvkmngz }
  q-pjme: { nid: 1795356732666, hash: f3fzxqc4, sync: 53zddjur }
---

> AKA [[Identity provider]]

An **IdP (Identity Provider)** is the authoritative service in an [[Identity federation]] or SSO architecture that verifies a principal's identity ([[Authentication]]) and issues signed tokens ([[JWT]] via [[OIDC]]) or assertions ([[SAML]]) consumed by downstream applications (Service Providers / Relying Parties).

## Core Responsibilities

- **Credential & MFA verification**: Passwords, passkeys/WebAuthn, hardware tokens, adaptive risk checks.
- **Token & assertion issuance**: Signs tokens with rotating asymmetric keys published via JWKS (`/.well-known/jwks.json`) or SAML metadata.
- **Directory & lifecycle management**: User provisioning/deprovisioning (SCIM), group/role membership, session revocation.
- **Examples**: [[Keycloak]], Okta, Microsoft Entra ID, Auth0, AWS Cognito.

In federated auth, the ==Identity Provider (IdP)== authenticates the user and issues assertions/tokens, while the ==Service Provider (SP)== trusts those assertions to grant access to resources.
^q-xspc

> [!tip] Production reality
> Because every login and token refresh depends on the IdP, treat it as Tier-0 infrastructure. Downstream services and [[API Gateway]] instances should cache JWKS public keys locally with graceful refresh on unknown `kid` so a brief IdP blip does not break verification of already-issued [[JWT]]s.

Cache the IdP's JWKS public keys locally at your API gateway/services (with bounded refresh on unknown `kid`) so an IdP outage does not immediately take down active API traffic. #card-reminder
^q-pjme

## Related

- [[Identity provider]]
- [[Identity federation]]
- [[Keycloak]]
- [[OIDC]]
- [[SAML]]
- [[JWT]]
- [[Authentication]]
- [[Zero-Trust Security]]
