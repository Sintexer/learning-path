> AKA **OpenID Connect**

**OpenID Connect (OIDC)** is an [[Authentication|authentication]] layer built on top of **OAuth 2.0**. While OAuth 2.0 handles delegated [[Authorization|authorization]] (granting an app permission to call an [[API]]), OIDC standardizes how a client (Relying Party) verifies *who* the user is via an [[Identity provider|IdP]] (such as [[Keycloak]]).

What is the difference between OAuth 2.0 and OIDC?::OAuth 2.0 is a delegated authorization framework (issuing access tokens for APIs), whereas OIDC adds an authentication layer on top of OAuth 2.0 using an ID Token (JWT) and UserInfo endpoint.

## Core Artifacts & Discovery

- **ID Token**: A signed [[JWT]] asserting the user's identity (`sub`, `iss`, `aud`, `exp`, `nonce`, `auth_time`). Intended strictly for the **client application** that initiated login.
- **Access Token**: Credential sent to Resource Servers ([[API]]s) to authorize requests. May be a [[JWT]] or an opaque reference token—clients must treat it as opaque.
- **Refresh Token**: Long-lived credential used to obtain fresh access/ID tokens without re-prompting the user.
- **Discovery & JWKS**: Clients auto-configure endpoints and public signing keys via `/.well-known/openid-configuration` and `jwks_uri`.

## Recommended Flow

For browser SPAs and mobile apps, always use ==Authorization Code Flow with PKCE== instead of the deprecated Implicit flow.

1. Client generates a high-entropy `code_verifier` and its SHA-256 `code_challenge`, then redirects the user to the [[Identity provider|IdP]] with `state` (CSRF protection) and `nonce` (ID Token replay protection).
2. User authenticates at the IdP; IdP redirects back with a short-lived authorization `code`.
3. Client exchanges `code` + `code_verifier` (and `client_secret` for confidential backends) on a back-channel [[HTTPS]] call to receive tokens.

> [!warning] Failure modes & trade-offs
> - **ID Token vs Access Token misuse**: Never pass an ID Token as a `Bearer` token to downstream APIs (it carries client `aud`, not API scopes), and never parse an Access Token in the frontend to determine user identity.
> - **Confused deputy (missing `aud` check)**: If Resource Server B fails to verify the `aud` claim, a token minted for Service A can be replayed against Service B.
> - **Redirect URI hijacking**: Wildcard or open `redirect_uri` matching lets attackers exfiltrate authorization codes. Always enforce exact URI matching.
> - **Single Sign-Out friction**: Front-channel logout (hidden iframes) breaks under modern browsers' third-party cookie blocking; prefer back-channel logout or short token TTLs.

Always validate `iss`, `aud`, `exp`, and signature on every resource server to prevent confused-deputy token replay across services. #card-reminder

## Why must you never send an OIDC ID Token to a backend API as a Bearer token? #card

An ID Token is minted for the **client application** (`aud` = client ID) to prove user authentication—not to authorize API calls. Sending it to a backend API breaks audience isolation, bypasses API-specific scopes/permissions, and opens confused-deputy vulnerabilities where a token meant for one client is accepted by unrelated APIs. Always send an **Access Token** scoped to the target resource server.

## Related

- [[JWT]]
- [[SAML]]
- [[Identity provider]]
- [[Identity federation]]
- [[Keycloak]]
- [[Authentication]]
- [[Authorization]]
- [[API Gateway]]
