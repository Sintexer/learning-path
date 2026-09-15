**Kafka has zero awareness of consumer progress when deciding what to delete.** Retention is purely mechanical, configured per-[[Kafka Topic|topic]], based on either:

- **Time** — e.g., `retention.ms = 7 days` → anything older than 7 days gets deleted, period.
- **Size** — e.g., `retention.bytes = 10GB` → once a partition exceeds this size, oldest segments get deleted, period.

Kafka does not check "does any consumer group still need this?" It just deletes according to the policy. If slow consumer hasn't caught up yet and the messages it needs have already aged out past retention, here's what happens in practice: when that consumer tries to fetch an offset that no longer exists (because it was deleted), Kafka can't serve it. The consumer then falls back to whatever `auto.offset.reset` is configured to (`earliest` or `latest`) — meaning **Consumer silently jumps forward and permanently loses those messages it never got to.**

**This is a real production failure mode** — "slow consumer falls behind, retention expires, consumer jumps ahead and silently drops data." This is exactly the kind of practical gotcha interviewers love, so lock this in: **retention and consumption are completely decoupled.** Long retention + slow consumers is a real operational risk you're expected to know about.