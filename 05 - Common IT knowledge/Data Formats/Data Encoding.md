---
flashcards:
  q-sggv: { nid: 1795356729220, hash: dtrekj6c, sync: 78js2hbv }
---

Programs usually work with data in (at least) two different representations:
1. In memory, data is kept in objects, structs, lists, arrays, hash tables, trees, and so on. These [[Data Structures|data structures]] are optimized for efficient access and manipulation by the CPU (typically using pointers).
2. When you want to write data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (for example, a JSON document). Since a pointer wouldn’t make sense to any other process, this sequence-of-bytes representation looks quite different from the data structures that are normally used in memory.

The translation from the in-memory representation to a byte sequence is called **encoding** (also known as serialization or marshalling), and the reverse is called **decoding** (parsing, deserialization, unmarshalling).

## Built-in language formats

Many programming languages come with built-in support for encoding in-memory objects into byte sequences. For example, Java has `java.io.Serializable`, Ruby has `Marshal`, Python has `pickle`, and so on. Many third-party libraries also exist, such as Kryo for Java. But such formats are incompatible between languages, have bad support for schema evolution, and are not very optimized. Moreover, such an approach usually has to instantiate arbitrary classes on the fly, which raises serious security concerns (arbitrary code execution on untrusted input). They are usually temporary debug utilities, not production-ready solutions.

## Textual formats

JSON, XML, and CSV are the most widely known formats. They are widely supported, human readable, agile, and easy to use and extend. But they lack a generalized schema implementation, take too much space even in binary variants, and have ambiguity in some types (e.g., `int` vs `double` vs `long` in JSON — JavaScript numbers lose precision above $2^{53}$, forcing 64-bit IDs to be encoded as strings).

## Binary formats

For data that is used only internally within your organization, there is less pressure to use a lowest-common-denominator encoding format. For example, you could choose a format that is more compact or faster to parse. For a small dataset, the gains are negligible, but once you get into the terabytes, the choice of data format can have a big impact.

JSON is less verbose than XML, but both still use a lot of space compared to binary formats. This observation led to the development of a profusion of binary encodings for JSON (MessagePack, BSON, BJSON, UBJSON, BISON, and Smile, to name a few) and for XML (WBXML and Fast Infoset, for example). These formats have been adopted in various niches, but none of them are as widely adopted as the textual versions of JSON and XML.

Still, text-to-binary formats have drawbacks and still take more space than schema-based binary approaches. E.g., the lack of a schema forces these formats to keep field names in every encoded record.

Schema-based binary formats have several benefits:
- Schema can be shared once and not exchanged in every message. That makes messages lighter compared to text-to-binary formats.
- Field names are no longer required to be included per-message. However, each field value is identified by a **field tag** — a numerical alias for a field.
- Schema provides code generation (used by [[gRPC]] and RPC frameworks).
- Storage of versioned schemas acts as self-documentation (often managed via a Schema Registry in [[Event-driven architecture]]).

But there are also some drawbacks:
- It is harder to integrate external services or debug payloads manually without the schema.
- Managing schema evolution requires careful versioning and compatibility checks.

The most well-known and widely used binary schema-based formats are:
- [[Apache Thrift]]
- [[Protocol Buffers]]
- [[Apache Avro]]

What about [[Backward Compatibility]]? As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning. The only detail is that if you add a new field, you cannot make it *required*. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added. Therefore, to maintain backward compatibility, every field you add after the initial deployment of the schema must be optional or have a default value.

Removing a field is just like adding a field, with backward and [[Forward Compatibility]] concerns reversed. That means you can only remove a field that is *optional* (a required field can never be removed), and you can never use the same tag number again (because you may still have data written somewhere that includes the old tag number, and that field must be ignored by new code).

> [!warning] Failure modes & trade-offs
> - **Language-native deserialization RCE**: Deserializing untrusted bytes with `java.io.Serializable` or Python `pickle` can instantiate arbitrary gadget chains and execute attacker code. Never expose native serialization on network boundaries.
> - **JSON 64-bit integer truncation**: Passing 64-bit IDs (e.g., Snowflake IDs) as JSON numbers silently corrupts them in JS/browser clients (IEEE-754 doubles only represent integers exactly up to $2^{53} - 1$). Serialize 64-bit IDs as strings in [[REST]] APIs.
> - **Avro vs Thrift/Protobuf**: Avro omits field tags completely and relies on matching *writer schema* and *reader schema* by field name — ideal for dynamically generated database/Hadoop schemas and [[Change Data Capture]] streams, whereas Thrift/Protobuf excel at statically typed RPC contracts.

## Why should you avoid language-native serialization (e.g. `java.io.Serializable`, Python `pickle`) in production? #card
1. Ties data to one programming language.
2. Poor [[Backward Compatibility|backward]] and [[Forward Compatibility|forward]] schema evolution support.
3. Severe security risk: deserializing untrusted input can instantiate arbitrary classes (RCE).
4. Often suboptimal CPU/space efficiency compared to dedicated binary formats.

When removing a field in a tag-based binary schema (Protobuf/Thrift), you can only remove an ==optional== field and you must ==never reuse its tag number==.

Why are 64-bit integers often encoded as strings in JSON APIs?::JavaScript/IEEE-754 numbers lose integer precision above 2^53 - 1, silently corrupting large IDs.
^q-sggv

## Related
- [[Backward Compatibility]]
- [[Forward Compatibility]]
- [[Protocol Buffers]]
- [[Apache Thrift]]
- [[gRPC]]
- [[REST]]
- [[DDIA Chapter 4 Encoding and Evolution]]
