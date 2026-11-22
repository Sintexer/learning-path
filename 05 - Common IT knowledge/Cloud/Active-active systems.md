---
flashcards:
  q-dguk: { nid: 1795356730493, hash: vummqkke, sync: ddgteayk }
  q-he8v: { nid: 1795356730519, hash: vcggwd86, sync: yrfr7cxf }
  q-ag83: { nid: 1795356730544, hash: v9jj325e, sync: 3eyypgya }
  q-mpg6: { nid: 1795356730570, hash: k2hzpm4p, sync: pnq2p3pz }
---

A disadvantage of [[Active-passive systems]] is scalability. This is where an active-active system shines. With both servers [[Availability|available]], the second server can take some load for the application, and the entire system can take more load. However, if the application is stateful, there would be an issue if the customer's session isn't available on both servers. Stateless applications work better for active-active systems, moving shared state into an external store such as [[Redis]] or [[Memcached]]. Suitable for [[Horizontal scaling]].

In multi-region active-active setups, every region serves live traffic simultaneously through global [[Load Balancing]] or geo-routed [[DNS]]. Because writes can arrive in multiple regions at once, data stores must use [[Multi-Leader Replication]] or [[Leaderless Replication]] (or partition users cleanly by home region).

> [!warning] Failure modes & trade-offs
> - **Write conflicts & [[Replication Lag]]**: concurrent updates to the same record in two regions require conflict resolution (LWW, [[CRDT|CRDTs]], or regional affinity).
> - **Headroom math (N+1 / 50% cap)**: if two active regions run above 50% capacity each, losing one region overloads the survivor and causes a cascading outage unless you enforce [[Rate Limiting]] and [[Backpressure]].
> - **[[Split-Brain]]**: network partitions between active nodes can cause divergent state if you lack [[Quorum Consensus]] or [[Fencing Token|fencing tokens]].

## Flashcards

Active-active architecture::A high-availability topology where multiple nodes or regions serve live traffic simultaneously behind a [[Load Balancing|load balancer]].
^q-dguk

For an active-active deployment to survive losing one of two regions without cascading failure, each region must run below ==50%== capacity (or shed load via [[Rate Limiting]]).
^q-he8v

Always externalize session state before moving from active-passive to active-active so instances stay disposable and horizontally scalable. #card-reminder
^q-ag83

### Why do stateful applications struggle in active-active topologies? #card

Because subsequent requests from the same client can land on different instances; unless session state is externalized (e.g., [[Redis]]) or pinned with sticky sessions, requests fail or see stale state.
^q-mpg6

## Related
- [[Active-passive systems]]
- [[Availability]]
- [[Horizontal scaling]]
- [[Load Balancing]]
- [[Multi-Leader Replication]]
- [[Split-Brain]]
