> AKA **Uniform Resource Identifier**

A **URI** is a string of characters used for identifying a resource or name on the Internet (RFC 3986). A URI can be a [[URL|locator (URL)]], a **name (URN)**, or **both**, and it consists of:

- **Scheme**: identifies the protocol or namespace (e.g., `https`, `ftp`, `urn`, `mailto`)
- **Hierarchical / Specific part**: authority (`host:port`), path, optional query (`?k=v`), and optional fragment (`#section`)

> [!example]
> - URL (locator + identifier): `https://www.example.com/myFolder/myFile`
> - URN (name only, no location): `urn:isbn:978-1449373320`

![[URL#Difference between URI and URL]]

## Flashcards

URI:::Uniform Resource Identifier — a string that uniquely identifies a resource by location ([[URL]]), by name (URN), or both.

Give an example of a URI that is *not* a [[URL]].::A URN such as `urn:isbn:978-1449373320` (identifies a book by name without specifying how or where to fetch it).

## Related
- [[URL]]
- [[HTTP]]
- [[REST]]
- [[DNS]]
