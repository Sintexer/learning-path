The security **principle of least privilege ([[POLP]])** means giving a user or system only those privileges ([[Authorization]]) that are essential to perform its intended function. For example, a user who works in supporting databases does not need to manage the network. By applying this principle you can protect your application from worst-case scenarios when credentials are lost or compromised: even with stolen credentials, an intruder will fail to steal bank information if the only thing they have is weather service permissions.

## Core Practices

1. **Grant access as needed** — Grant access to data and other resources to only the people or services that really need that access. Start with **default-deny** and grant access as needed based on job role or service boundary.
2. **Enforce separation of duties** — Enforcing separation of duties with appropriate [[Authorization|authorization]] for each interaction with your resources is a security best practice. Set clear boundaries on how authority is delegated across software engineers, operations staff, and other job functions (e.g., the engineer who writes a deployable artifact should not be the sole approver who pushes it to production).
3. **Avoid long-term credentials** — Centralize privilege management and reduce or eliminate reliance on static, long-term credentials. Prefer temporary, auto-expiring credentials (e.g., [[OIDC]] workload identity, STS assume-role tokens, short-lived [[JWT]]s) to limit exposure if credentials leak.

> [!warning] Failure modes & trade-offs
> - **Permission creep during incidents**: Engineers frequently add wildcard (`*`) permissions to unblock an outage and forget to tighten them afterward. Enforce permission boundaries and automated IAM drift/access-analyzer checks.
> - **Over-scoping shared service accounts**: Running multiple [[Microservices architecture|microservices]] under one shared IAM role or [[Kubernetes]] ServiceAccount defeats least privilege—one compromised service inherits the union of all permissions. Give every service its own identity.
> - **Productivity friction**: Overly rigid manual approval flows encourage shadow workarounds; pair least privilege with automated Just-In-Time (JIT) elevation for break-glass access.

What is the primary security benefit of the Principle of Least Privilege (POLP)?::It minimizes the **blast radius** when a user account, workload, or credential is compromised—an attacker can only reach the minimal resources that identity was explicitly granted.

## Why should every microservice have its own IAM identity / ServiceAccount instead of sharing one? #card

Sharing a single IAM role or [[Kubernetes]] ServiceAccount across services grants every service the **union** of all permissions. A single vulnerability (e.g., SSRF or RCE) in a low-sensitivity service then exposes high-sensitivity databases, secrets, or queues owned by other services—violating [[Zero-Trust Security]] and least privilege.

Start all authorization policies with ==default-deny==, enforce ==separation of duties==, and replace static credentials with ==short-lived temporary tokens==.

## Related

- [[POLP]]
- [[Authorization]]
- [[Zero-Trust Security]]
- [[Kubernetes]]
- [[SSE]]
- [[OIDC]]
