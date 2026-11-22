---
flashcards:
  q-y4x3: { nid: 1795356724598, hash: mugawbya, sync: 43nfgvr7 }
  q-swix: { nid: 1795356724620, hash: ki3f69t4, sync: 37brphza }
---

## Eventual consistency

**[[Eventual Consistency|Eventual consistency]]** is a data consistency model in distributed computing where updates to a piece of data propagate asynchronously across replicas ([[Replication]]), guaranteeing that all read requests will eventually return the last updated value if no new updates are made.

## East-West vs North-South Traffic

- **East-West Traffic**: Data flow *between* servers, [[Virtual machine|virtual machines]], or [[Container|containers]] inside a single data center, [[Virtual Private Cloud|VPC]], or [[Kubernetes]] cluster (service-to-service calls, database queries, replication).
- **North-South Traffic**: Data flow *entering or leaving* the data center or cluster across its perimeter (external clients hitting an [[API Gateway]], [[Load Balancing|load balancer]], or [[Reverse proxy]], and outbound calls to external APIs).

East-West Traffic:::Internal service-to-service data flow inside a data center, VPC, or cluster (as opposed to North-South perimeter ingress/egress traffic).
^q-y4x3

> [!tip] Production reality
> In [[Microservices architecture|microservices]], East-West traffic frequently exceeds North-South traffic by 10x or more (a single user request fans out into dozens of internal RPCs). Perimeter firewalls alone cannot protect it—this is why [[Zero-Trust Security]], mTLS via a [[Service Mesh]], and [[Distributed Tracing]] ([[OpenTelemetry]]) are essential inside the cluster.

Why is perimeter-only security insufficient in a microservices cluster?::Because most traffic is East-West (service-to-service inside the cluster); once an attacker breaches one pod, lateral movement is unrestricted unless East-West traffic enforces mTLS and service-level authorization (Zero-Trust).
^q-swix

## Related

- [[Eventual Consistency]]
- [[Microservices architecture]]
- [[API Gateway]]
- [[Service Mesh]]
- [[Zero-Trust Security]]
- [[Distributed Tracing]]
- [[OpenTelemetry]]
- [[Virtual Private Cloud]]
