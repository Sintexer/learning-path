
## Reliability

> Reliability means making systems work correctly, even when faults occur.

Reliability is a property of a system that tells how well it could continue to operation correctly even when things go wrong.

Working correctly means it operates in a way user expects it to be.

Reliable system can handle user and software failures, is performative in a normal load conditions, and prevents unauthorized access and abuse.

The things that can go wrong are called **faults**, and systems that anticipate faults and can cope with them are called **fault-tolerant** or resilient. Fault is a deviation of a one component from the spec. **Failure** is a type of fault that is mostly unrecoverable - application stops operating in the way the user expects it to.

**Fault types**:
- **Hardware faults** - common on large scale systems. Hard drive failure, RAM malfunction, power outage etc. Handled by data centers. Reliability allows even upgrading the system using [[Zero-downtime deployment#Rolling update|Rolling update]].
- **Software Errors** - bugs hiding in application code for some time and executing when unusual conditions met. Can cause function failure, or cascading failures. Application code should be written resiliently to avoid failing on unexpected errors.
- **Human errors** - most common type. Usually caused by misconfiguration, or accidental operation on prod env.

## Scalability

> Scalability means having strategies for keeping performance good, even when load increases.

Scalability is a term used yo describe if system is capable of coping with increasing load. It is not a one-dimensional label: it would be wrong to say “X is scalable” or “Y doesn’t scale.” Rather, discussing scalability means considering questions like “If the system grows in a particular way, what are our options for coping with the growth?” and “How can we add computing resources to handle the additional load?”

To discuss scalability one would need to measure the **load** in some way. Load can be described with a few numbers which we call *load parameters*. The best choice of parameters depends on the architecture of your system:
- requests per second to a web server, 
- the ratio of reads to writes in a database, 
- the number of simultaneously active users in a chat room, 
- the hit rate on a cache, 
- or something else. 

Perhaps the average case is what matters for you, or perhaps your bottleneck is dominated by a small number of extreme cases.

> [!tip] Tweeter's scenario
> Tweeter has two interconnected features: tweet post and users' home timeline. In v1 posting a tweet would just update the tweets table and trigger all subscribers to rebuild the timeline when they open it. That caused massive read overload, as building a home timeline required heavy tables join operations. Then, in v2, twitter shifted to a different model - now each tweet write caused home timeline update for all subscribers, which decreases the timeline rebuild time drastically. That optimized performance a lot for average users (~75 followers). However accounts int tweeter are distributed unevenly and some celebrities might have 30M+ followers. Now tweeter spends a lot of resources on tweet write. Fun fact is they are moving to a hybrid approach to account for large accounts rebuild problem.

In a batch processing system such as Hadoop, we usually care about **throughput**—the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.

In online systems, what’s usually more important is the service’s **response time**—that is, the time between a client sending a request and receiving a response.

> [!info] Latency and response time
> **Latency** and **response time** are often used synonymously, but they are not the same. The response time is what the client sees: besides the actual time to process the request (the service time), it includes network delays and queueing delays. Latency is the duration that a request is waiting to be handled—during which it is *latent*, await‐ ing service

Response time is usually measured as average. But it is little useful - most requests might complete fast, but it is the slow requests that are causing the most troubles for users. That is why a common way to measure response time is to use `hdrHistograms` or percentiles. Percentile 99 shows the actual response time 1% of your users receive. Working on improving the 99.9 percentile could improve the experience for 1 of 1000 requests. Amazon targets this percentile because they calculated that this 1 of 1000 request costs them 1% of revenue. But optimizing such small percentiles is a hard work, because sometimes slow requests are caused by random hardware events and so on.

Often response time latencies are caused by something called *head of line blocking*. Relatively lightweight and fast request can spend a huge amount of time waiting for the server resource just because several prior slow requests are taking all available threads.

### Approaches for Coping with Load

How do we maintain good performance even when our load parameters increase by some amount?


Some systems are **elastic**, meaning that they can automatically add computing resources when they detect a load increase, whereas other systems are scaled manually (a human analyzes the capacity and decides to add more machines to the system). An elastic system can be useful if load is highly unpredictable, but manually scaled systems are simpler and may have fewer operational surprises.

The architecture of systems that operate at large scale is usually highly specific to the application—there is no such thing as a generic, *one-size-fits-all* scalable architecture (informally known as magic scaling sauce). The problem may be:
- the *volume of reads*,
- *the volume of writes*, 
- *the volume of data to store*, 
- *the complexity of the data*, 
- *the response time requirements*, 
- *the access patterns*, 
- or (usually) *some mixture* of all of these plus many more issues.

For example, a system that is designed to handle 100,000 requests per second, each 1 kB in size, looks very different from a system that is designed for 3 requests per minute, each 2 GB in size—even though the two systems have the same data throughput.

An architecture that scales well for a particular application is built around assumptions of which operations will be common and which will be rare — the **load parameters**. If those assumptions turn out to be wrong, the engineering effort for scaling is at best wasted, and at worst counterproductive. In an early-stage startup or an unproven product it’s usually more important to be able to iterate quickly on product features than it is to scale to some hypothetical future load.

## Maintainability

> Maintainability has many facets, but in essence it’s about making life better for the engineering and operations teams who need to work with the system.

Every system grows and evolves. However the majority of the software cost is contained not in the initial development part, but in its ongoing maintenance. To minimize the cost of the maintenance, we should design the software in the way it adhere to the three principles:
1. **Operability** - Make it easy for operations teams to keep the system running smoothly. Monitoring health, keeping software up-to-date, maintaining knowledge base, anticipating future problems.
2. **Simplicity** - Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (Note this is not the same as simplicity of the user interface.) 
3. **Evolvability** - Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change. Also known as *extensibility*, *modifiability*, or *plasticity*.

Good abstractions can help reduce complexity and make the system easier to modify and adapt for new use cases. Good operability means having good visibility into the system’s health, and having effective ways of managing it.