---
flashcards:
  q-2afv: { nid: 1795361722229, hash: jfapcehx, sync: pferfhji }
---

In **key-range partitioning**, each partition owns a continuous range of keys from a minimum to a maximum (like volumes of a printed encyclopedia). Boundaries do not need to be evenly spaced; they are chosen to distribute data volume evenly (see [[Rebalancing Partitions#Dynamic Partitioning|Dynamic Partitioning]]).

Within each partition, keys are kept in **sorted order** (for example inside an [[SSTable]] or [[B-Tree Index|B-Tree]]), making range scans (`WHERE key BETWEEN a AND b`) fast and localized to one or a few partitions.

## Write Skew on Monotonic Keys

Sorting by key makes range queries fast, but monotonic keys cause severe [[Partitioning Hot Spots and Skew|write skew]]. For example, if the primary key is a `timestamp` from IoT sensors, every write for the entire day hits the single partition owning "today" while all other partitions sit idle.

To avoid this, **prefix the timestamp** with a high-cardinality dimension such as `sensor_id` (`(sensor_id, timestamp)`). Writes spread across all sensors in parallel, while each sensor's data stays sorted chronologically. The trade-off is that querying across *all* sensors for a time window now requires separate range queries per sensor.

> [!warning] Failure modes & trade-offs
> - **Hot-tail writes**: Auto-incrementing IDs, ULIDs, UUIDv7, and raw timestamps all append to the rightmost range partition, turning a 10-node cluster into a 1-node write bottleneck.
> - **Cold-start single node**: Under dynamic range splitting, a brand-new empty table starts with one partition on one node until it grows past the split threshold unless you **pre-split** key boundaries upfront.

## How does key-range partitioning enable fast range queries, and what is its main write-path risk? #card
Each partition stores a contiguous range of keys in sorted order, so range scans hit only the owning partition(s) sequentially. The main risk is **write hot spots** when keys increase monotonically (such as timestamps or sequential IDs), concentrating all new writes on a single partition.

How can you avoid timestamp write hot spots in a key-range partitioned sensor database?::Prefix the timestamp with a high-cardinality attribute (e.g., `(sensor_id, timestamp)`) so writes spread across partitions, accepting that cross-sensor time queries require one range scan per sensor.
^q-2afv

## Related
- [[Data partitioning]]
- [[Hash Partitioning]]
- [[Partitioning Hot Spots and Skew]]
- [[Rebalancing Partitions]]
- [[Partitioning and Secondary Indexes]]
- [[SSTable]]
