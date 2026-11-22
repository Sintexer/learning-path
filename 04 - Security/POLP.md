> AKA **Principle of least privilege**

![[Least privilege principle]]

## Applying POLP in Production

- **Default-deny policies**: Start with zero permissions and grant only explicit actions on explicit resource ARNs/scopes (never `Action: "*"` or `Resource: "*"` in production).
- **Short-lived workload identities**: Replace static API keys and long-lived IAM credentials with [[OIDC]] workload identity (e.g., AWS IRSA, GCP Workload Identity, SPIFFE/SPIRE in [[Kubernetes]]).
- **Just-In-Time (JIT) elevation**: Eliminate standing production admin access; require time-boxed, audited elevation for break-glass tasks.
- **Service-to-service isolation**: Pair [[Authorization]] policies with mTLS in a [[Service Mesh]] as part of [[Zero-Trust Security]].

> [!tip] Production reality
> Over-broad permissions usually creep in during urgent incident debugging ("temporarily" granting wildcard access) and are never revoked. Automate permission boundary checks and IAM access-analyzer reviews in CI/CD to catch privilege drift.

Replace static long-lived secrets with short-lived workload identities (e.g., OIDC / IRSA / SPIFFE) so leaked credentials expire automatically. #card-reminder

What is Just-In-Time (JIT) privilege elevation and how does it support POLP?::Instead of granting standing admin rights, JIT grants elevated permissions only for a short, audited time window when needed, minimizing blast radius if an account is compromised.

## Related

- [[Least privilege principle]]
- [[Authorization]]
- [[Zero-Trust Security]]
- [[Kubernetes]]
- [[Service Mesh]]
- [[OIDC]]
