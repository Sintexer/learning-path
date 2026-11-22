---
flashcards:
  q-8wcq: { nid: 1795356729320, hash: bfn7jxnh, sync: kmyfc5d9 }
  q-qmmy: { nid: 1795356729341, hash: kiyg4htc, sync: x8m34qgg }
---

Born at Facebook, **Apache Thrift** is a schema-based binary [[Data Encoding|encoding]] and RPC framework with a cross-language Interface Definition Language (IDL) and code generation tools.

It defines three binary wire formats:
- **BinaryProtocol** — straightforward binary encoding using field type annotations and numeric field tags instead of field names.
- **CompactProtocol** — packs field type and tag delta into a single byte and uses variable-length integers (varints) for smaller payloads.
- **DenseProtocol** — C++-only format that omits field tags on the wire and relies on identical schemas on both ends (sacrifices schema evolution flexibility).

```thrift
struct Person {
	1: required string userName,
	2: optional i64 favoriteNumber,
	3: optional list<string> interests
}
```

## Schema Evolution

Like [[Protocol Buffers]], Thrift identifies fields by numeric **field tags** rather than field names:
- Adding a new field preserves [[Backward Compatibility]] only if the field is `optional` or has a default value; older code skips unknown tags for [[Forward Compatibility]].
- Removing a field is safe only for `optional` fields, and removed tag numbers must never be reused.
- Changing field names is safe because names are not sent on the wire; changing a field's data type risks truncation or decoding errors.

> [!warning] Failure modes & trade-offs
> Marking fields `required` in Thrift creates a permanent deployment hazard: you can never remove a `required` field without breaking older readers, and you can never add a new `required` field without breaking newer readers on old data. Prefer `optional` fields and validate invariants in application code instead.

## How do Thrift's BinaryProtocol and CompactProtocol differ? #card
- **BinaryProtocol** encodes fixed-width integers and separate type + tag bytes, favoring simplicity and parsing speed.
- **CompactProtocol** uses variable-length integers (varints) and packs field type and tag delta into one byte, producing significantly smaller payloads at a slight CPU cost.

In [[Apache Thrift]], changing a field's name is safe on the wire because ==fields are identified by numeric tag numbers, not field names==.
^q-8wcq

Never reuse a deleted field tag in Thrift or Protobuf; stale stored data or in-flight messages with the old tag will silently decode with the wrong type or meaning. #card-reminder
^q-qmmy

## Related
- [[Data Encoding]]
- [[Protocol Buffers]]
- [[Backward Compatibility]]
- [[Forward Compatibility]]
- [[DDIA Chapter 4 Encoding and Evolution]]
