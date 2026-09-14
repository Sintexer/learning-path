Programs usually work with data in (at least) two different representations:
1. In memory, data is kept in objects, structs, lists, arrays, hash tables, trees, and so on. These data structures are optimized for efficient access and manipulation by the CPU (typically using pointers).
2. When you want to write data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (for example, a JSON document). Since a pointer wouldn’t make sense to any other process, this sequence-of-bytes representation looks quite different from the data structures that are normally used in memory.

The translation from the in-memory representation to a byte sequence is called **encoding** (also known as serialization or marshalling), and the reverse is called **decoding** (parsing, deserialization, unmarshalling).

## Built-in language formats

Many programming languages come with built-in support for encoding in-memory objects into byte sequences. For example, Java has java.io.Serializable, Ruby has Marshal, Python has pickle, and so on. Many third-party libraries also exist, such as Kryo for Java. But such formats are incompatible between languages, have bad support for schema evolution, not very optimized. Moreover such approach usually has to construct an arbitrary classes on the fly, which raises security concerns. They are usually some temp debug utilities, not a prod-ready solutions.

## Textual formats

Json, XML, and CSV are the most widely known formats. They are widely supported, human readable, agile, ease to use and extend. But they lack generalized schema implementation, take too much space even in binary format, there are a lot of ambiguity in some types (e.g. int vs double vs long in json).

## Binary formats

For data that is used only internally within your organization, there is less pressure to use a lowest-common-denominator encoding format. For example, you could choose a format that is more compact or faster to parse. For a small dataset, the gains are negligible, but once you get into the terabytes, the choice of data format can have a big impact.

JSON is less verbose than XML, but both still use a lot of space compared to binary formats. This observation led to the development of a profusion of binary encodings for JSON (MessagePack, BSON, BJSON, UBJSON, BISON, and Smile, to name a few) and for XML (WBXML and Fast Infoset, for example). These formats have been adopted in various niches, but none of them are as widely adopted as the textual versions of JSON and XML.

Still, text-to-binary formats have drawbacks and still take more space than schema based binary approaches. E.g. the lack of schema forces these formats to keep field names in data.

Schema based binary formats have several benefits:
- Schema could be shared once and not exchanged during the communication process. That makes messages lighter compared to text-to-binary formats.
- Field names are no longer required to be included per-message. owever each field value is identified by **field tag** - a numerical alias for a field.
- Schema provides code generation
- Storage of versioned schemas could act as a self-documentation

But also there are some drawbacks:
- It is much harder to integrate services. If schemas are incompatible, communication is stopped. Unlike json, which just could ignore unknown fields.
- Managing schemas evolution requires careful versioning.


The most well known and widely-used binary schema-based formats are:
- [[Apache Thrift]]
- [[Protocol Buffers]]
- [[Apache Avro]]

What about [[Backward Compatibility]]? As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning. The only detail is that if you add a new field, you cannot make it *required*. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added. Therefore, to maintain backward compatibility, every field you add after the initial deployment of the schema must be optional or have a default value.

Removing a field is just like adding a field, with backward and [[Forward Compatibility]] concerns reversed. That means you can only remove a field that is *optional* (a required field can never be removed), and you can never use the same tag number again (because you may still have data written somewhere that includes the old tag number, and that field must be ignored by new code).