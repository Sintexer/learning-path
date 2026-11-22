---
flashcards:
  q-3mwn: { nid: 1795356732842, hash: w5bkgxhi, sync: 8pdmxmx3 }
  q-k5sp: { nid: 1795356732867, hash: mu3fpash, sync: mthsas26 }
---

> **Authentication answers the question, "Are you who you say you are?" Authorization answers the question, "What actions can you perform?"**

![[Authentication]]

![[Authorization]]

## Quick Comparison

| Dimension | [[Authentication]] (AuthN) | [[Authorization]] (AuthZ) |
| :--- | :--- | :--- |
| Question | Who are you? | What are you allowed to do? |
| Order | Always happens first | Happens after identity is established |
| [[HTTP]] status on failure | `401 Unauthorized` (unauthenticated) | `403 Forbidden` (authenticated, lacks permission) |
| Common protocols / models | [[OIDC]], [[SAML]], mTLS, Passkeys | OAuth 2.0 scopes, RBAC, ABAC, ReBAC |

`HTTP 401 Unauthorized` means ==the caller is not authenticated (missing or invalid credentials)==, whereas `HTTP 403 Forbidden` means ==the caller is authenticated but lacks permission for the requested action==.
^q-3mwn

> [!tip] Production reality
> Sometimes returning `404 Not Found` instead of `403 Forbidden` is intentional: if a user requests `/invoices/999` owned by another tenant, returning `403` confirms that invoice `999` exists (existence oracle). Returning `404` hides resource existence from unauthorized callers.

Return `404 Not Found` instead of `403 Forbidden` when confirming a resource's existence to an unauthorized caller would leak sensitive information. #card-reminder
^q-k5sp

## Related

- [[Authentication]]
- [[Authorization]]
- [[JWT]]
- [[OIDC]]
- [[SAML]]
- [[Identity provider]]
- [[Least privilege principle]]
- [[Zero-Trust Security]]
