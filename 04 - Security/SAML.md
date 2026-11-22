---
flashcards:
  q-tvyg: { nid: 1795356731969, hash: akrqrjr5, sync: y93rp5s7 }
  q-6ebz: { nid: 1795356731992, hash: f2gt3eyw, sync: c8ef7uxe }
  q-xgyu: { nid: 1795356732017, hash: dyrgb3tu, sync: 26pnpsmn }
---

**SAML 2.0 (Security Assertion Markup Language)** is an XML-based open standard for [[Identity federation]] and enterprise Single Sign-On (SSO). It allows an [[Identity provider|IdP]] (e.g., Okta, Microsoft Entra ID, [[Keycloak]]) to assert a user's [[Authentication|authentication]] state and attributes to a **Service Provider (SP)** without sharing passwords.

## Core Flows

1. **SP-Initiated SSO (Recommended)**:
   - User visits the SP (application).
   - SP generates a signed `AuthnRequest` (with a unique request ID) and redirects the browser to the IdP.
   - User authenticates at the IdP; IdP returns a Base64-encoded, XML-signed `SAMLResponse` assertion via HTTP-POST to the SP's **Assertion Consumer Service (ACS)** URL.
   - SP verifies the XML signature, `InResponseTo`, `Audience`, and `NotBefore`/`NotOnOrAfter` timestamps, then creates a local session.
2. **IdP-Initiated SSO**:
   - User clicks an app tile inside the IdP portal; IdP posts an unsolicited `SAMLResponse` directly to the SP's ACS URL.

Why is SP-initiated SAML SSO safer than IdP-initiated SSO?::SP-initiated SSO ties the IdP's assertion to a specific request ID (`InResponseTo`), mitigating login CSRF and unsolicited replay attacks.
^q-tvyg

## SAML 2.0 vs OIDC

| Dimension | SAML 2.0 | [[OIDC]] |
| :--- | :--- | :--- |
| Format | XML + XMLDSig | JSON / [[JWT]] |
| Transport | Browser redirects / HTTP-POST forms | [[REST]] / [[HTTPS]] |
| Primary sweet spot | Enterprise B2B SSO, legacy directories | Mobile apps, SPAs, [[API]]s, [[Microservices architecture|microservices]] |
| API authorization | Not designed for API delegation | Built on OAuth 2.0 access tokens |

> [!warning] Failure modes & trade-offs
> - **XML Signature Wrapping (XSW) & XXE**: XML parsing is complex and historically error-prone. Always use hardened SAML libraries that disable external entity resolution (XXE) and verify the signature covers the exact `<Assertion>` node the application reads.
> - **Certificate rotation outages**: IdP signing certificates expire. If the SP only holds one cert at a time instead of supporting overlapping dual-cert metadata rotation, cert expiry causes an instant company-wide SSO outage.
> - **Assertion replay**: Enforce `NotBefore`/`NotOnOrAfter` with bounded clock skew and cache seen assertion IDs until expiry.

An ==XML Signature Wrapping (XSW)== attack exploits discrepancies between which XML element the signature verifier checks and which unsigned element the application actually reads.
^q-6ebz

## When should you choose SAML 2.0 vs OIDC? #card

- Choose **SAML 2.0** when integrating with enterprise B2B customers whose corporate IdPs (ADFS, Okta, Entra ID) mandate SAML for workforce SSO.
- Choose **OIDC** for greenfield web/mobile apps, SPAs, and [[Microservices architecture|microservices]] where lightweight JSON/[[JWT]] tokens and OAuth 2.0 API authorization are needed.
^q-xgyu

## Related

- [[OIDC]]
- [[Identity federation]]
- [[Identity provider]]
- [[Keycloak]]
- [[Authentication]]
- [[JWT]]
- [[HTTPS]]
