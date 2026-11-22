---
flashcards:
  q-ay8b: { nid: 1795356730291, hash: maim76fg, sync: esn7tfin }
  q-ctb7: { nid: 1795356730317, hash: v397kvfs, sync: viea5iag }
  q-5awx: { nid: 1795356730342, hash: yf9av5jh, sync: qkza832t }
  q-3ekv: { nid: 1795356730368, hash: upmvw6jp, sync: v9jr3cn9 }
---

The availability of a system is typically expressed as a percentage of uptime in a given year or as a number of nines. In the following table is a list of availability percentages based on the downtime per year and its notation in nines.

| **Availability (%)**                            | **Downtime (per year)** |
| ----------------------------------------------- | ----------------------- |
| 90% (one nine of availability)                  | 36.53 days              |
| 99% (two nines of availability)                 | 3.65 days               |
| 99.9% (three nines of availability)             | 8.77 hours              |
| 99.95% (three and a half nines of availability) | 4.38 hours              |
| 99.99% (four nines of availability)             | 52.60 minutes           |
| 99.995% (four and a half nines of availability) | 26.30 minutes           |
| 99.999% (five nines of availability)            | 5.26 minutes            |

To increase availability, you need redundancy. This typically means more infrastructure—more data centers, more servers, more databases, and more [[Replication|replication of data]]. You can imagine that adding more of this infrastructure means a higher cost. Customers want the application to always be available, but you need to draw a line where adding redundancy is no longer viable in terms of revenue.

Two ways to increase availability are [[Active-active systems]] and [[Active-passive systems]].

## Composite Availability Math

- **Components in series** (every dependency must succeed): \(A_{\text{total}} = A_1 \times A_2 \times \dots \times A_n\). Five 99.9% synchronous dependencies in a chain drop overall availability to \(0.999^5 \approx 99.5\%\).
- **Components in parallel** (redundant replicas, any one suffices): \(A_{\text{total}} = 1 - (1 - A_1)(1 - A_2)\). Two independent 99.9% replicas yield \(1 - 0.001^2 = 99.9999\%\) (assuming no shared failure domain).

> [!tip] Production reality
> Adding another "nine" roughly multiplies engineering and infrastructure cost by an order of magnitude. Above 99.99%, human intervention is too slow to stay within the error budget (~52 min/year), so automated failover, [[Health Checks]], [[Circuit Breaker|circuit breakers]], and [[Zero-downtime deployment|zero-downtime deployments]] become mandatory. Always distinguish **SLI** (measured metric), **SLO** (internal target), and **SLA** (external contractual commitment).

## Flashcards

How much annual downtime corresponds to 99.9% (three nines) and 99.99% (four nines) availability?::99.9% ≈ 8.77 hours/year; 99.99% ≈ 52.6 minutes/year.
^q-ay8b

For components in series, composite availability is the ==product== of individual availabilities; for independent components in parallel, it is ==1 minus the product of their unavailability==.
^q-ctb7

Every extra nine of availability costs roughly 10x more; match your SLO to business impact rather than chasing five nines by default. #card-reminder
^q-5awx

### Why does chaining many synchronous microservices hurt availability? #card

Because synchronous dependencies are in series: overall availability multiplies (\(A_1 \times A_2 \times \dots \times A_n\)). Even ten 99.9% services in a synchronous chain yield only ~99.0% availability unless decoupled asynchronously or protected by [[Fallback Pattern|fallbacks]].
^q-3ekv

## Related
- [[Active-active systems]]
- [[Active-passive systems]]
- [[Reliability, Scalability, Maintainability]]
- [[Health Checks]]
- [[Circuit Breaker]]
- [[Fallback Pattern]]
