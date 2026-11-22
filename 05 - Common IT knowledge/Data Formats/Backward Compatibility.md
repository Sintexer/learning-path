---
flashcards:
  q-qcit: { nid: 1795356729245, hash: ucxigkhk, sync: em8by753 }
  q-w56p: { nid: 1795356729267, hash: z963bmec, sync: ence8wbt }
  q-swti: { nid: 1795356729291, hash: 3t6xtke9, sync: yaz2ywju }
---

In contrast to [[Forward Compatibility]], **backward compatibility** means newer code can read and modify data that was written by older code.

Backward compatibility is normally not hard to achieve: as author of the newer code, you know the format of data written by older code, and so you can explicitly handle it (if necessary by simply keeping the old code to read the old data).

In terms of how [[Data Encoding#Binary formats]] (such as [[Protocol Buffers]] and [[Apache Thrift]]) support backward compatibility:
- As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning.
- The only detail is that if you add a new field, you cannot make it required. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added. Therefore, to maintain backward compatibility, ==every field added after initial deployment must be optional or have a default value==.
^q-qcit

> [!tip] Production reality
> Data almost always outlives code. Rows written five years ago in a database or archived events in [[Kafka]]/object storage may remain untouched until a migration or replay reads them with today's binary. Backward compatibility is therefore a long-term invariant, not just a rolling-upgrade concern.

Backward compatibility:::Newer code can read data written by older code.
^q-w56p

## Why must you support both backward and forward compatibility during a rolling upgrade? #card
During a [[Zero-downtime deployment#Rolling update|rolling upgrade]], old and new versions run concurrently: new nodes must read data written by old nodes (backward compatibility), and old nodes must safely read data written by new nodes (forward compatibility).
^q-swti

## Related
- [[Forward Compatibility]]
- [[Data Encoding]]
- [[Protocol Buffers]]
- [[Apache Thrift]]
- [[API versioning]]
- [[Schema Migration]]
- [[DDIA Chapter 4 Encoding and Evolution]]
