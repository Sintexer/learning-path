---
flashcards:
  q-3r2j: { nid: 1795356732568, hash: 2fk3q2nm, sync: 6vpzp43y }
  q-r4xx: { nid: 1795356732592, hash: gbckw9v9, sync: sds99k8b }
  q-r5dp: { nid: 1795356732616, hash: ahyscxfw, sync: edvmbkg4 }
---

**Identity federation** allows users with credentials managed elsewhere—like a corporate directory or external [[Identity provider]]—to get access to a resource. E.g., allowing users from LDAP or Okta to access an application via [[Keycloak]] using [[SAML]] or [[OIDC]].

Identity federation is a system of trust between two parties for the purpose of authenticating users ([[Authentication]]) and conveying information needed to authorize ([[Authorization]]) their access to resources. In this system, an [[Identity provider|IdP]] is responsible for user authentication, and a **Service Provider (SP)**—such as a service or an application—controls access to resources.

## The Flow

1. A **trust relationship** is configured between the [[Identity provider|identity provider]] and the service provider (exchanging signing certificates / JWKS endpoints and client/entity metadata). The service provider trusts the identity provider to [[Authentication|authenticate users]] and relies on the claims provided by the identity provider.
2. After authenticating a user, the identity provider returns a signed message—called an **assertion** ([[SAML]]) or **ID token** ([[OIDC]] / [[JWT]])—containing the user's identifier and attributes needed to establish a session and determine the scope of resource access.
3. The service provider receives and validates the cryptographic signature, issuer (`iss`), audience (`aud`), and expiration timestamps, then issues local session credentials or access tokens.
4. With the right credentials from the service provider, the user now has direct access to the requested resources via an established session.

> [!warning] Failure modes & trade-offs
> - **Attribute/group-mapping privilege escalation**: If the SP trusts raw group names (e.g., `admin`) from an external federated IdP without tenant scoping, a misconfigured or compromised external IdP could assert admin privileges across tenants. Always scope federated claims to the specific tenant/IdP.
> - **Account takeover via email linking**: Auto-linking federated accounts to existing local accounts solely by unverified `email` claims lets an attacker hijack accounts. Link on immutable `(iss, sub)` pairs and require verified emails.
> - **Stale deprovisioning**: Federated login only runs at sign-in; without SCIM provisioning or short session TTLs, a terminated employee may retain access until their SP session expires.

In identity federation, never auto-link external accounts solely by `email`—key users by immutable ==(issuer, subject) / `(iss, sub)`== pairs.
^q-3r2j

## What is the difference between an Identity Provider (IdP) and a Service Provider (SP) in identity federation? #card

- **Identity Provider (IdP)**: Owns user credentials, performs authentication (including MFA), and issues cryptographically signed assertions/tokens ([[SAML]] or [[OIDC]]).
- **Service Provider (SP)**: Trusts the IdP's signing keys, validates the assertion/token, and enforces local [[Authorization]] over its own resources.
^q-r4xx

Always scope role/group claims from an external federated IdP to that tenant so a misconfigured external directory cannot escalate privileges across tenants. #card-reminder
^q-r5dp

## Related

- [[Identity provider]]
- [[IdP]]
- [[OIDC]]
- [[SAML]]
- [[Keycloak]]
- [[JWT]]
- [[Authentication]]
- [[Authorization]]
- [[Zero-Trust Security]]
