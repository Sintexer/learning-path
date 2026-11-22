---
flashcards:
  q-4w7h: { nid: 1795256794749, hash: uk9aziqx, sync: 2t5aiiyi }
  q-h7w6: { nid: 1795356732691, hash: b7k26hpd, sync: 72mgc3mh }
  q-9dbp: { nid: 1795356732721, hash: amv4ztwg, sync: wrnaf82t }
  q-3zyz: { nid: 1795356732743, hash: tbw9unxd, sync: yjj6qhm9 }
---

## Authorization

After you’re authenticated and in your account, you might be curious about *what actions you can take*. This is where authorization comes in. **Authorization** is the process of giving users permission to access AWS resources and services. **Authorization** determines whether a user can perform certain actions, such as read, edit, delete, or create resources. ==**Authorization** answers the question, “What actions can you perform?”==
^q-4w7h

## Authorization Models

- **RBAC (Role-Based Access Control)**: Permissions are grouped into roles (`admin`, `editor`, `viewer`) assigned to users. Simple and auditable, but struggles with fine-grained ownership rules (role explosion).
- **ABAC (Attribute-Based Access Control)**: Policies evaluate attributes of the subject, resource, action, and environment (e.g., `user.department == doc.department && time < 18:00`). Flexible, harder to audit statically.
- **ReBAC (Relationship-Based Access Control)**: Models permissions as a graph of relationships (e.g., Google Zanzibar / SpiceDB: `user:alice -> member -> group:eng -> viewer -> doc:42`). Ideal for collaborative, hierarchical multi-tenant apps.

## Where to Enforce Authorization

- **Coarse-grained** checks (valid [[JWT]], active account, broad `scope`/role) belong at the [[API Gateway]].
- **Fine-grained / object-level** checks ("does user X own order Y?") must be enforced inside the owning service against authoritative data.

> [!warning] Failure modes & trade-offs
> - **BOLA / IDOR (Broken Object Level Authorization)**: Checking only that a user is authenticated ([[Authentication]]) or has role `customer`, without verifying they own the requested resource ID (`/orders/123`), is consistently the #1 API vulnerability.
> - **Stale claims in JWTs**: Baking fine-grained permissions into long-lived [[JWT]] tokens means revoking a user's access does not take effect until the token expires.
> - **Fail-open bugs**: Always design authorization middleware to **default-deny** ([[Least privilege principle]]); if a policy lookup times out or throws, deny the request rather than allowing it through.

What is Broken Object Level Authorization (BOLA / IDOR)?::A vulnerability where an endpoint verifies authentication or coarse role, but fails to check whether the caller is authorized to access the specific resource ID requested (e.g., changing `/orders/101` to `/orders/102`).
^q-h7w6

## Compare RBAC, ABAC, and ReBAC authorization models. #card

- **RBAC**: Assigns permissions via static roles; simplest to reason about, prone to role explosion for per-resource rules.
- **ABAC**: Evaluates dynamic policies over user/resource/environment attributes; expressive, harder to audit who can access what.
- **ReBAC**: Evaluates graph relationships between principals and resources (e.g., Zanzibar); excels at nested folders, orgs, and shared documents.
^q-9dbp

Authorization checks must always fail closed (default-deny) and enforce object-level ownership inside the owning service, not just at the API gateway. #card-reminder
^q-3zyz

## Related

- [[Authentication and Authorization]]
- [[Authentication]]
- [[Least privilege principle]]
- [[POLP]]
- [[JWT]]
- [[OIDC]]
- [[API Gateway]]
- [[Zero-Trust Security]]