---
flashcards:
  q-jy4f: { nid: 1795356720668, hash: afhyqpkm, sync: ekprr2xj }
  q-b5vb: { nid: 1795356720693, hash: ejzqw99m, sync: yfv887z2 }
  q-ya68: { nid: 1795356720716, hash: a6tbu36i, sync: r3ynwyu3 }
---

## Reliability

> Reliability means making systems work correctly, even when faults occur.

Reliability is a property of a system that tells how well it continues to operate correctly even when things go wrong.

Working correctly means it operates in the way the user expects it to.

A reliable system can handle user and software failures, performs well under normal load conditions, and prevents unauthorized access and abuse.

The things that can go wrong are called **faults**, and systems that anticipate faults and can cope with them are called **fault-tolerant** or resilient. A **fault** is a deviation of one component from its spec. A **failure** is when the system as a whole stops operating in the way the user expects it to.

**Fault types**:
- **Hardware faults** — common on large-scale systems. Hard drive failure, RAM malfunction, power outage, etc. Handled via redundancy and multi-machine software fault tolerance. Reliability also allows upgrading systems without downtime using [[Zero-downtime deployment#Rolling update|rolling updates]].
- **Software errors** — systematic bugs hiding in application code until unusual conditions trigger them. Often correlated across nodes, causing cascading failures. Mitigated with isolation, [[Circuit Breaker|circuit breakers]], [[Bulkhead Pattern|bulkheads]], and thorough testing.
- **Human errors** — the most common cause of outages. Usually caused by misconfiguration or accidental operations in production. Mitigated with sandboxes, fast rollbacks, automated checks, and clear observability.

## Scalability

> Scalability means having strategies for keeping performance good, even when load increases.

Scalability is a term used to describe a system's ability to cope with increasing load. It is not a one-dimensional label: it would be wrong to say “X is scalable” or “Y doesn’t scale.” Rather, discussing scalability means considering questions like “If the system grows in a particular way, what are our options for coping with the growth?” and “How can we add computing resources to handle the additional load?”

To discuss scalability one must measure **load** with a few numbers called *load parameters*. The best choice of parameters depends on the architecture of your system:
- requests per second to a web server,
- the ratio of reads to writes in a database,
- the number of simultaneously active users in a chat room,
- the hit rate on a cache,
- or something else (e.g., fan-out distribution skew).

Perhaps the average case is what matters for you, or perhaps your bottleneck is dominated by a small number of extreme cases.

> [!tip] Twitter's fan-out scenario
> Twitter has two interconnected features: posting a tweet and reading a user's home timeline. In v1, posting a tweet only inserted into the tweets table and every follower's timeline read ran a heavy join across all followed accounts — causing massive read overload. In v2, Twitter shifted to **fan-out-on-write**: each tweet write pushed into a pre-computed home timeline cache for every follower, making reads fast for average users (~75 followers). However, follower counts are heavily skewed and celebrities have 30M+ followers, turning a single tweet write into a massive write storm. Twitter adopted a **hybrid approach**: fan-out-on-write for normal users, fan-out-on-read merged at query time for celebrities.

In a batch processing system such as Hadoop, we usually care about **throughput** — the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.

In online systems, what’s usually more important is the service’s **response time** — the time between a client sending a request and receiving a response.

> [!info] Latency and response time
> **Latency** and **response time** are often used synonymously, but they are not the same. **Response time** is what the client sees: besides the actual time to process the request (service time), it includes network delays and queueing delays. **Latency** is the duration that a request is waiting to be handled — during which it is *latent*, awaiting service.

Response time is often reported as an average, which is misleading — most requests may complete fast while a slow minority ruins the experience for high-value users. Instead, measure **percentiles** (e.g., p50, p95, p99, p99.9) using tools like `HdrHistogram`. Percentile 99 shows the response time threshold that 99% of requests beat (1% are slower). Amazon targets p99.9 because their slowest 1-in-1000 requests belong to users with the most data/orders and directly impact revenue.

Often tail-latency spikes are caused by **head-of-line blocking**: a lightweight, fast request spends a huge amount of time waiting in a queue just because a few slow requests occupy all available server threads/connections (see [[Little's Law]]).

> [!warning] Failure modes & trade-offs
> **Tail latency amplification**: When a single user request fans out to $N$ backend services in parallel and waits for all of them, even a small chance of a slow backend call gets multiplied. If each backend has a 1% chance of being slow (p99), fanning out to 100 backends means $1 - 0.99^{100} \approx 63\%$ of end-user requests will hit a p99 slowdown. Mitigate with [[Timeout Pattern|timeouts]], [[Backpressure]], [[Rate Limiting]], and hedged requests (sending a duplicate request if the first is slow, for [[Idempotency|idempotent]] reads).

### Approaches for Coping with Load

How do we maintain good performance even when our load parameters increase? We combine [[Vertical scaling|scaling up (vertical)]] and [[Horizontal scaling|scaling out (horizontal)]] (via [[Load Balancing]], [[Replication]], and [[Data partitioning]]).

Some systems are **elastic**, meaning they automatically add computing resources when they detect a load increase, whereas other systems are scaled manually. An elastic system is useful if load is highly unpredictable, but manually scaled systems are simpler and have fewer operational surprises.

The architecture of systems that operate at large scale is usually highly specific to the application — there is no generic, *one-size-fits-all* scalable architecture. The bottleneck may be:
- the *volume of reads*,
- *the volume of writes*,
- *the volume of data to store*,
- *the complexity of the data*,
- *the response time requirements*,
- *the access patterns*,
- or (usually) *some mixture* of all of these.

For example, a system designed to handle 100,000 requests per second of 1 KB each looks very different from a system designed for 3 requests per minute of 2 GB each — even though both have the same data throughput.

An architecture that scales well is built around assumptions of which operations will be common and which will be rare — the **load parameters**. If those assumptions turn out to be wrong, the engineering effort for scaling is at best wasted and at worst counterproductive.

## Maintainability

> Maintainability has many facets, but in essence it’s about making life better for the engineering and operations teams who need to work with the system.

Every system grows and evolves, and the majority of software cost is in ongoing maintenance rather than initial development. To minimize maintenance cost, design for three principles:
1. **Operability** — Make it easy for operations teams to keep the system running smoothly (monitoring health, [[Distributed Tracing]] / [[OpenTelemetry]], keeping software up-to-date, maintaining runbooks, anticipating future problems).
2. **Simplicity** — Make it easy for new engineers to understand the system by removing accidental complexity through clean abstractions. (Note: this is not the same as UI simplicity.)
3. **Evolvability** — Make it easy for engineers to change the system for unanticipated future requirements (also known as *extensibility*, *modifiability*, or *plasticity*; closely tied to [[DDIA Chapter 4 Encoding and Evolution]]).

## What is the difference between a fault and a failure? #card
- **Fault**: One component deviates from its specification (e.g., a disk dies, a packet is dropped).
- **Failure**: The system as a whole stops providing the required service to the user.
Fault-tolerant systems prevent component faults from escalating into system-wide failures.
^q-jy4f

## What is tail latency amplification and why does it matter in microservices? #card
When a user request fans out to multiple backend calls in parallel and must wait for all of them, the overall response time is dictated by the slowest single call. Even if each call has only a 1% chance of being slow, fanning out to 50–100 calls makes hitting tail latency the common case for end users.

**Response time** is ==the total time the client waits (service time + network delay + queueing delay)==, whereas **latency** is ==the duration a request waits in a queue awaiting service==.
^q-b5vb

Never call a system "scalable" without specifying the load parameters it scales along and the latency percentiles it must hold. #card-reminder
^q-ya68

## Related
- [[Availability]]
- [[Scaling]]
- [[Horizontal scaling]]
- [[Vertical scaling]]
- [[Load Balancing]]
- [[Little's Law]]
- [[Backpressure]]
- [[Rate Limiting]]
- [[Zero-downtime deployment]]
- [[OpenTelemetry]]
- [[Designing Data Intensive Applications]]
