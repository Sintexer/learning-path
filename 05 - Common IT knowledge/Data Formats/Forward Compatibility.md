In contrast to [[Backward Compatibility]], **forward compatibility** means older code can read data that was written by newer code.

Forward compatibility is generally harder than backward compatibility because it requires older code to gracefully ignore additions made by a newer version of the schema that did not exist when the older code was written.

## Why It Matters in Production

- **[[Zero-downtime deployment#Rolling update|Rolling upgrades]]**: During a rolling deploy, new and old service instances run simultaneously. A newer instance may write a record to a database or publish an event to [[Kafka]] that an older instance then reads.
- **Shared databases**: A database often acts as a communication channel between service versions. Older instances reading rows updated by newer instances must not crash on newly added fields.
- **Client upgrades**: Mobile or desktop clients running older versions may receive responses from a newly deployed backend.

## Rules in Schema-Based Binary Formats

In tag-based formats like [[Protocol Buffers]] and [[Apache Thrift]]:
- **Adding a field**: Safe for forward compatibility — older code encounters an unrecognized field tag, uses the wire-type length annotation to skip it, and continues decoding.
- **Removing a field**: You can only remove an *optional* field; removing a `required` field breaks older code that expects it.
- **Changing a field type**: Risky — e.g., widening a 32-bit integer to 64-bit may cause older code to truncate the value.

> [!warning] Failure modes & trade-offs
> **Read-modify-write data loss**: Even if older code can *read* a record written by newer code by ignoring unknown fields, if the older code then *updates and writes back* that same record without preserving unknown fields, it can silently wipe out the new fields written by the newer version.

Forward compatibility:::Older code can read data written by newer code.

## What subtle bug happens when older code performs a read-modify-write on data written by newer code? #card
If the older code parses the record into an in-memory object that lacks the newly added fields and then writes the object back, it silently strips and loses the new fields unless unknown fields are explicitly preserved.

In [[Data Encoding#Binary formats|tag-based binary formats]], older code achieves forward compatibility when reading new fields by ==skipping unrecognized field tags using their wire-type length==.

## Related
- [[Backward Compatibility]]
- [[Data Encoding]]
- [[Protocol Buffers]]
- [[Apache Thrift]]
- [[Schema Migration]]
- [[Zero-downtime deployment]]
- [[DDIA Chapter 4 Encoding and Evolution]]
