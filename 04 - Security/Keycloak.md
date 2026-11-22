**Keycloak** is an open-source Identity and Access Management (IAM) server and [[Identity provider|IdP]] (built on Quarkus) that provides Single Sign-On (SSO), [[Authentication]], [[Authorization]], and [[Identity federation]] out of the box using [[OIDC]], OAuth 2.0, and [[SAML]] 2.0.

## Core Building Blocks

- **Realms**: Isolated security domains with their own users, roles, signing keys, sessions, and clients.
- **Clients**: Applications or services that request authentication or tokens (`public` clients like SPAs/mobile using PKCE; `confidential` backend clients using a client secret or signed JWT).
- **Roles & Groups**: **Realm roles** (global across the realm) vs **Client roles** (scoped to a specific service).
- **Client Scopes & Protocol Mappers**: Control which claims/roles are embedded into issued [[JWT]] tokens.
- **User Federation vs Identity Brokering**:
  - **User Federation**: Connects Keycloak to an external user store (LDAP / Active Directory) to look up and sync users.
  - **Identity Brokering**: Delegates login to an external [[Identity provider|IdP]] (Google, GitHub, corporate SAML/OIDC IdP).

In Keycloak, what is the difference between User Federation and Identity Brokering?::User Federation connects Keycloak to an external user directory (like LDAP/Active Directory) to look up users, whereas Identity Brokering delegates login to an external IdP (like Google, GitHub, or a corporate SAML/OIDC provider).

> [!warning] Failure modes & trade-offs
> - **SPOF on the auth path**: Run Keycloak in an HA cluster backed by [[PostgreSQL]] (never embedded H2) with properly tuned Infinispan distributed/persistent sessions—if Keycloak goes down, new logins and token refreshes halt.
> - **Realm sprawl in multi-tenant SaaS**: One realm per customer breaks down past a few hundred realms due to memory, cache, and startup overhead. For large B2B SaaS, prefer shared realms with Organizations/groups/tenant claims.
> - **JWT bloat**: Mapping all realm/client roles into every access token bloats headers past proxy limits (4–8 KB). Scope tokens per audience via Client Scopes and [[Least privilege principle]].
> - **Console drift**: Manual edits in the Admin UI cause dev/prod divergence; codify realms, clients, and mappers via [[IaaC]] (e.g., [[Terraform]] Keycloak provider).

Manage Keycloak realms, clients, and mappers via IaC (e.g., Terraform) rather than manual Admin Console edits to prevent environment drift. #card-reminder

## Why is creating one Keycloak Realm per customer an anti-pattern at high tenant counts? #card

Each Keycloak realm maintains its own keys, caches, event listeners, and metadata loaded into memory. Beyond a few hundred realms, startup times, memory footprint, and admin/migration operations degrade sharply. At scale, use a shared realm with Keycloak Organizations, tenant attributes, or group-based isolation instead.

## Related

- [[Identity provider]]
- [[Identity federation]]
- [[OIDC]]
- [[SAML]]
- [[JWT]]
- [[Authentication]]
- [[Authorization]]
- [[API Gateway]]
- [[Terraform]]
