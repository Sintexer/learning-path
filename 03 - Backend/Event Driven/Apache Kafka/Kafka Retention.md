---
flashcards:
  q-en92: { nid: 1795356733241, hash: qq83i4ks, sync: z9d9yqnk }
---

**Kafka has zero awareness of consumer progress when deciding what to delete.** Retention is purely mechanical, configured per-[[Kafka Topic|topic]], based on either:

- **Time** — e.g., `retention.ms = 7 days` → anything older than 7 days gets deleted, period.
- **Size** — e.g., `retention.bytes = 10GB` → once a partition exceeds this size, oldest segments get deleted, period.

Kafka does not check "does any consumer group still need this?" It just deletes according to the policy. If slow consumer hasn't caught up yet and the messages it needs have already aged out past retention, here's what happens in practice: when that consumer tries to fetch an offset that no longer exists (because it was deleted), Kafka can't serve it. The consumer then falls back to whatever `auto.offset.reset` is configured to (`earliest` or `latest`) — meaning **Consumer silently jumps forward and permanently loses those messages it never got to.**

**This is a real production failure mode** — "slow consumer falls behind, retention expires, consumer jumps ahead and silently drops data." This is exactly the kind of practical gotcha interviewers love, so lock this in: **retention and consumption are completely decoupled.** Long retention + slow consumers is a real operational risk you're expected to know about.

> [!tip] Production reality
> - **Monitor consumer lag in time, not just offset count:** Alert when consumer lag approaches a fraction of `retention.ms` (e.g. > 25% of retention window) so on-call can intervene (scale consumers, fix downstream [[Backpressure]], or temporarily extend `retention.ms`) before data ages out.
> - **Tiered Storage (KIP-405):** Offloads older log segments to cheap object storage (S3/GCS), allowing long retention windows (months/years) for replay/audit without scaling expensive local broker NVMe disks.
> - **Hybrid policy (`cleanup.policy=compact,delete`):** Compacts keys while still bounding total storage by deleting keys older than `retention.ms`.

## Flashcards

## What happens when a slow [[Kafka Consumer]] falls behind a topic's retention window? #card
Because Kafka retention is completely decoupled from consumer progress, aged-out segments are deleted regardless of consumer lag. When the consumer requests the deleted offset, Kafka cannot serve it and the consumer falls back to `auto.offset.reset` (`earliest` or `latest`), silently skipping and permanently losing the expired messages.

Kafka retention is configured per [[Kafka Topic]] by time (`retention.ms`) or size (`retention.bytes`) and has ==zero awareness of consumer progress==.
^q-en92

## Related
- [[Kafka]]
- [[Kafka Topic]]
- [[Kafka Partition]]
- [[Kafka Consumer]]
- [[Event Sourcing]]
- [[Backpressure]]