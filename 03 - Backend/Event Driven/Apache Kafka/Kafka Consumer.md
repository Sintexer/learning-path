Consumer receives the data from the broker's topic (identified by name) using the *pull model*. Consumers automatically know which broker to read from. In case of broker failures, consumers know how to recover. 

Data is read from low to high offset **within each partition**.

Kafka stores the offsets at which a consumer group has been reading. The offsets committed are in Kafka *topic* names `__consumer_offsets`. When a consumer has processed data received from Kafka, it should be periodically committing the offsets (the Kafka broker will write to `__consumer_offsets`, not the consumer group itself).

## Consumer groups

A **consumer group** is just a label — a `group.id` string that a consumer process declares when it connects. Kafka uses this label for exactly one purpose: **tracking a single shared "reading position" (offset) per partition, for all consumers that share that label.**

Two consumers with the **same** `group.id` → they _split the work_. Kafka assigns each of them a subset of the topic's partitions, so each message is handled by exactly **one** consumer within that group. This is how you scale out processing.

Two consumers with **different** `group.id`s → they are **completely invisible to each other**. Each group gets its own independent bookmark on the same topic. Group A could be sitting at offset 10,000 on partition 0, while Group B — reading that exact same partition — is sitting at offset 200. Kafka doesn't care. It just serves whatever offset each group asks for.

> [!tip]
> The same physical stream of messages can be consumed by many different services, each fully independently, each at its own pace, each remembering its own progress — with zero coordination between those services required at runtime.

All the consumers in an application read data as a *consumer group*. Each consumer within a group reads from exclusive partitions. Each partition is read by one consumer in a consumer group. If there are more consumers than partitions, they should be split into more consumer groups.

**Kafka does not know what a "service" is.** It has no concept of service identity, deployment, container, or codebase. All Kafka sees is: a TCP connection from a client that, during initialization, declares a `group.id` string. That's it. `group.id` is just a string you put in a config file or connection setup. Kafka blindly trusts whatever string shows up.

This means:

**✅ Correct, intended use — horizontal scaling of ONE logical service:**  
Say Service A is deployed as 3 replicas (e.g., 3 pods in Kubernetes, or 3 instances behind a load balancer — though LB doesn't matter here, they connect directly to Kafka). All 3 replicas run **identical code** and all declare `group.id = "service-a-group"`. Kafka sees 3 consumers with the same group id, and splits the topic's partitions across them — e.g., if the topic has 6 partitions, each replica gets 2. This is the normal, healthy pattern for scaling out processing of the same logic. All 3 are "the same service" from a business logic perspective, just multiple running copies.

**⚠️ Dangerous, accidental misuse — two DIFFERENT services sharing a group.id:**  
This is the trap. Suppose someone building Service B copy-pastes config from Service A and forgets to change `group.id`. Now Service B (different code, doing Action2) is in the _same_ consumer group as Service A (doing Action1). Kafka doesn't know or care that they're different codebases — it just sees "two more members of group service-a-group" and starts handing them partitions to split. The result: **some messages get routed to Service A's logic, and some identical-looking messages get routed to Service B's logic, essentially at random (whichever gets the partition)** — neither service sees the full stream, and behavior becomes nondeterministic and broken. This is a very real, very common bug in early microservices migrations.

### Offset commits

Each consumer, after processing message(s), sends a **commit** — basically a message that says "group X has now successfully processed up to offset N on partition P." This commit itself gets written into a special internal Kafka topic called `__consumer_offsets`.

When a consumer restarts, crashes and comes back, or a rebalance happens (partitions get reassigned among group members), Kafka looks up: "what's the last committed offset for this group on this partition?" and resumes reading from right after that point.

**Two ways to commit:**

- **Auto-commit** — Kafka client library commits automatically on a timer (e.g., every 5 seconds), regardless of whether you've actually _finished_ processing. Danger: if your process crashes mid-processing, right after an auto-commit fired but before you finished the actual work, that message is now considered "done" by Kafka and will never be redelivered — **silent data loss** from the app's perspective. Simple, but risky.
- **Manual commit** — your code explicitly calls "commit" only _after_ it has successfully finished processing the message (e.g., after the DB write succeeds). This is safer — worst case if you crash before committing, you reprocess that message again on restart (a duplicate), but you never silently skip work. This is why manual commit + idempotent processing is the standard senior-level pairing, and why "auto-commit" alone is considered naive in production systems.

### Delivery semantics

On reconnect, consumer will start reading from the last committed offset. So there are several option to choose the delivery semantics based on whether the consumer tolerant for repeated messages or not:
1. **At least once** - (usually preferred) offsets are committed after the message is processed. If the processing goes wrong, the message will be read again. This can result in duplicate processing of messages.
2. **At most once** - Offsets are committed as soon as messages are received. If the processing goes wrong, some messages will be lost.
3. **Exactly once** - For Kafka use Transactional API (easy with Kafka Streams API), or use External Systems workflows: use an idempotent consumer

### Configuring the initial group offset

A **brand new consumer group** has **no committed offset yet**, so you must explicitly decide where it starts. This is a **one-time deployment decision**, not an ongoing dependency. The exact start option also highly relates to the [[Kafka Retention]] and a known failure mode for a slow consumer.

1. **`earliest`** — start from the very beginning of the topic's retained history. Correct choice if consumer needs to reconstruct/catch up on everything that ever happened (full replay).
2. **`latest`** — start from "now," ignore all history, only see new messages going forward. Correct if consumer only cares about things happening from this point onward.
3. **Explicit offset/timestamp reset** — tell Kafka exactly: "set this group's offset to whatever corresponds to timestamp X" (using tooling like `kafka-consumer-groups.sh --reset-offsets --to-datetime ...`, or doing it programmatically). This is usually the _real_ answer in a migration scenario — you pick the exact moment the split happened, so consumer picks up cleanly from there without missing anything or reprocessing everything from the dawn of time.
### The Group Coordinator

For every consumer group, Kafka designates **one broker** to act as that group's **Group Coordinator**. This isn't a special separate server — it's just one of your regular Kafka brokers, elected to also handle this coordination role for a given group (different groups can have different coordinator brokers, spreading the load).

The Group Coordinator's job is narrow and mechanical:

- Track which consumers are currently "alive" and members of the group (via heartbeats)
- Facilitate the assignment process when membership changes (a **rebalance**)
- Store the finalized partition assignment and hand it out

### The actual protocol, step by step

1. **JoinGroup**: When a consumer instance starts up, it sends a `JoinGroup` request to the Group Coordinator, essentially saying "I exist, I want in on group X."
2. **Leader election (among consumers, not brokers)**: The coordinator picks one of the joining consumers to be the **group leader** for this rebalance round (this is just a role for this one operation — not a permanent thing, and totally separate from Kafka's broker leader/follower replication concept, unfortunately similar naming). This is usually just the first consumer that joined.
3. **The actual assignment computation happens on the _client side_**: This is the part that surprises people. The Group Coordinator does **not** decide who gets which partition. It sends the elected leader consumer the **full list of current group members** and topic metadata (partition counts, etc.). The leader consumer then runs a **partition assignment strategy algorithm locally**, in its own client library code, and computes the full assignment map: "consumer-1 gets partitions 0,1 — consumer-2 gets partitions 2,3 — consumer-3 gets partitions 4,5."
4. **SyncGroup**: The leader sends this computed assignment back to the coordinator via `SyncGroup`. The coordinator then distributes each consumer its specific assignment (each consumer only learns its own partitions, not necessarily the full map, though the leader computed the full thing).
5. Every consumer now knows its assigned partitions and starts fetching + processing from its last committed offset on those partitions.

**So**: consumers don't talk to each other directly, but they're not blind either — they coordinate _through_ the broker acting purely as a message relay and bookkeeper, while the actual assignment logic (the "smart" part) is computed by client-side library code (whichever consumer happens to be elected leader that round), not by the broker itself. The broker stays "dumb" in the sense that it never runs your assignment algorithm or understands _why_ a particular split makes sense — it just relays and stores the result.

### What determines the actual split (the algorithm itself)

This is configurable via `partition.assignment.strategy`, common options:

- **Range**: assigns contiguous partition ranges per consumer (simple, can be uneven if partition/consumer counts don't divide evenly)
- **RoundRobin**: spreads partitions evenly one at a time across consumers
- **Sticky**: tries to preserve previous assignments as much as possible when rebalancing, minimizing churn
- **CooperativeSticky** (modern default in recent Kafka versions): allows a rebalance to happen _incrementally_ — only the partitions that actually need to move get revoked and reassigned, rather than the old "stop-the-world" approach where **all** consumers in the group pause and give up all their partitions before reassignment (the old "eager" protocol). This matters practically because eager rebalancing caused a full processing pause for the whole group on every membership change, even minor ones — cooperative sticky minimizes that disruption.

### Heartbeats and detecting failure

Each consumer periodically sends a **heartbeat** to the Group Coordinator on a background thread (`heartbeat.interval.ms`). If the coordinator doesn't hear from a consumer within `session.timeout.ms`, it considers that consumer dead and triggers a rebalance to redistribute its partitions to the survivors.

Separately, there's `max.poll.interval.ms` — this guards against a consumer that's _alive_ (heartbeating fine) but stuck not actually calling `poll()` again (e.g., stuck processing a message too long). If it exceeds this interval without polling, it's kicked from the group too, even though heartbeats looked healthy — a subtlety that trips people up in production ("why did my slow-processing consumer get rebalanced away even though it was clearly alive?").