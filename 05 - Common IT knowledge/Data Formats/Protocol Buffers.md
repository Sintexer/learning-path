---
flashcards:
  q-iwzu: { nid: 1795356729192, hash: 9uwtinus, sync: yxx9g7nf }
---

**Protocol Buffers (Protobuf)** is a schema-based binary [[Data Encoding|encoding]] format developed by Google, paired with code generation tools (`protoc`) for strongly typed models across languages. It is the default wire format for [[gRPC]].

```protobuf
message Person {
	required string user_name = 1;
	optional int64 favorite_number = 2;
	repeated string interests = 3;
}
```

## Wire Format & Schema Evolution

Protobuf encodes each field as a `(field_number, wire_type)` key using variable-length integers (varints) rather than repeating field names on the wire.

- **Proto2 vs Proto3**: Proto3 removed the `required` keyword because required fields make long-term schema evolution brittle; all singular fields default to zero-values if omitted.
- **[[Backward Compatibility]]**: New fields must be optional (or have implicit defaults in proto3) so new code can read data written by older code.
- **[[Forward Compatibility]]**: Older code ignores unrecognized field tags and skips over their byte length using the wire type.
- **Safe removal**: Only optional fields can be removed, and their tag numbers should be marked `reserved` so nobody accidentally reuses them later.

> [!warning] Failure modes & trade-offs
> - **Silent default ambiguity in proto3**: Without explicit `optional` (presence tracking), a receiver cannot tell whether an `int64` field was explicitly set to `0` or omitted by an older sender.
> - **Tag reuse corruption**: Reusing a deleted tag number causes old stored records or in-flight messages to be misinterpreted under the new field's semantics.
> - **Unknown-field round-tripping**: If an older service decodes a message with unknown fields and re-encodes it to a database or downstream queue, it may drop those unknown fields unless unknown-field preservation is enabled.

## Why did Proto3 remove the `required` field modifier? #card
A `required` field can never be safely added (breaks [[Backward Compatibility]] when new code reads old data) and can never be safely removed (breaks [[Forward Compatibility]] when old code reads new data). Validating presence at the application layer avoids locking the wire schema forever.

What Protobuf keyword prevents accidental reuse of deleted field numbers?::`reserved`

In Proto3, omitting presence tracking (`optional`) creates ambiguity because ==an unset scalar field is indistinguishable from an explicit zero value==.
^q-iwzu

## Related
- [[Data Encoding]]
- [[Apache Thrift]]
- [[gRPC]]
- [[Backward Compatibility]]
- [[Forward Compatibility]]
- [[DDIA Chapter 4 Encoding and Evolution]]
