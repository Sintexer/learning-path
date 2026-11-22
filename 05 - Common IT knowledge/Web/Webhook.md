---
flashcards:
  q-99np: { nid: 1795356718941, hash: ist9fr8n, sync: pq5z9iva }
  q-9vc8: { nid: 1795361691230, hash: zpatfwkx, sync: wgxe3w63 }
  q-fnmc: { nid: 1795361691303, hash: fwbwcxi9, sync: eqx275rx }
  q-wpty: { nid: 1795356719067, hash: dgtdnm92, sync: f98srx5r }
---

## What is a webhook?

Assume we run an eCommerce website. Clients send orders to the order service via the [[API Gateway]], which calls the payment service. The payment service then talks to an external payment service provider (PSP) to complete the transaction.

There are two ways to receive asynchronous status updates from the external PSP:

**1. Short polling ([[Client polling pattern]])**

After sending the payment request to the PSP, the payment service keeps asking the PSP about the payment status until it completes.

Drawbacks:
- Constant polling wastes compute, network, and [[Rate Limiting|rate-limit]] quota on both sides.
- Introduces latency equal to the polling interval.

**2. Webhook**

We register a callback [[URL]] with the external service ("call me back at this URL when payment status changes"). When the PSP finishes processing, it sends an [[HTTPS]] `POST` request to our endpoint with the event payload.

Webhooks are often called **reverse APIs** or **push APIs** because the provider initiates the HTTP call to the consumer.

What if the PSP callback is lost or never arrives? Pair webhooks with a periodic **[[Reconciliation]]** job that polls the PSP for any payment stuck in `PENDING` state.

## Production Webhook Checklist (Receiver & Sender)

- **Signature verification**: verify an HMAC-SHA256 signature header (e.g., `X-Signature`) computed over the **raw request body** using a shared secret and constant-time comparison to prevent spoofing and timing attacks.
- **Replay protection**: include a signed timestamp in the signature payload and reject requests older than a tight window (e.g., 5 minutes), combined with event-ID deduplication.
- **[[Idempotency]]**: webhook providers guarantee [[At-least-once delivery]], so duplicate and out-of-order deliveries are normal—deduplicate by `event_id` ([[Idempotent Consumer]]) and guard state transitions ([[Version Guard]]).
- **Acknowledge fast, process async**: validate the signature, persist/enqueue the raw event (e.g., [[Kafka]] / DB queue), and return `200 OK` immediately. Doing slow business logic inline causes provider timeouts and retry storms.
- **Sender-side safety**: enforce strict [[Timeout Pattern|timeouts]], exponential backoff with jitter ([[Retry Pattern]]), [[DLQ]] for dead endpoints, and **SSRF protection** (block private RFC 1918 / cloud metadata `169.254.169.254` IPs when calling user-registered webhook URLs).

> [!warning] Failure modes & trade-offs
> - **Webhook retry storms**: if your receiver takes >5–10s to reply or returns `5xx` during a DB hiccup, providers retry with growing frequency, amplifying the outage—decouple ingestion from processing via a queue.
> - **Out-of-order events**: `payment.succeeded` can arrive *before* `payment.created` due to network retries; design handlers around state machines or fetch authoritative state from the provider API.
> - **Missing signature verification**: an unauthenticated webhook endpoint lets any attacker forge fake `payment.succeeded` events.

## Flashcards

Webhook:::A user-registered [[HTTPS]] callback [[URL]] that a provider invokes via `POST` when an asynchronous event occurs ("push/reverse API").
^q-99np

How do you protect a webhook receiver against spoofed requests and replay attacks?::Verify an ==HMAC signature== over the raw body + timestamp using constant-time comparison, reject timestamps outside a short window (e.g., 5 min), and deduplicate by `event_id` ([[Idempotent Consumer]]).
^q-jpyf
^q-9vc8

Why should a webhook endpoint enqueue the event and return `200 OK` immediately instead of running full business logic synchronously?::To prevent provider ==timeouts and retry storms== if downstream processing is slow or temporarily degraded.
^q-7n2e
^q-fnmc

### Why are webhooks alone not sufficient for critical workflows like payments? #card

Webhooks can be dropped, delayed, duplicated, or delivered out of order. Always combine webhook ingestion (for low-latency updates) with a periodic [[Reconciliation]] job that queries the provider API for stuck pending operations.
^q-wpty

## Related
- [[Client polling pattern]]
- [[Idempotency]]
- [[Idempotent Consumer]]
- [[Reconciliation]]
- [[At-least-once delivery]]
- [[API Gateway]]
- [[Retry Pattern]]
- [[DLQ]]
