Document Databases (such as [[MongoDB]]), a category of [[NoSQL database]]s, store data as collections of self-contained objects (JSON/BSON). Usually document databases have no strict schema (*schema-on-read*), which allows them to store different kinds of data in one collection and reduces [[Impedance mismatch]] for tree-structured aggregates (1-to-many nested data).

However such architecture is not suitable for many-to-one and many-to-many relationships. Document databases usually store such relationships as denormalized duplicates within the document. E.g. the city name has no id, it is just repeated across entries, causing problems with duplicates, complications of renaming a value (when city name changes due to politics), handling free-text input and mapping values to a same style (case, writing).

Even if the original design of the application's data model fits well into a document database, data has a tendency to change over time, and a need for joins might arise over time. If a database doesn't support joins, they will be executed on the application side. E.g. LinkedIn has references to schools and workplaces, which might have their own account. If these values are links to other entities, the app will automatically fetch related data and update it as it updates.

> [!warning] Failure modes & trade-offs
> - **Storage locality cuts both ways:** Storing a whole aggregate together gives single-seek reads, *only* if you typically read the whole document. Updating a single field inside a multi-megabyte document often requires rewriting the entire document on disk.
> - **Schema-on-read != schemaless:** The schema still exists — it just moved into your application code. Old document versions live alongside new ones forever unless you run background backfills, so readers must stay [[Backward Compatibility|backward compatible]].
> - **Convergence with [[RDBMS]]:** Modern relational databases like [[PostgreSQL]] (`jsonb` + [[GIN Index]]) now support nested JSON documents alongside full relational joins and [[ACID]] transactions.

## Flashcards

For what kind of data relationships do [[Document Database|document databases]] shine, and where do they struggle?::They shine for self-contained one-to-many tree structures (aggregates fetched as a whole), and struggle with many-to-one and many-to-many relationships that require joins or fan-out updates.

Why is "schemaless" a misleading description of [[Document Database|document databases]]?::They enforce **schema-on-read** rather than schema-on-write — the schema still exists implicitly in application code, which must handle multiple historical document shapes.

## Related

- [[MongoDB]]
- [[NoSQL database]]
- [[SQL vs NoSQL]]
- [[Relational Database]]
- [[Impedance mismatch]]
- [[Graph database]]
- [[PostgreSQL]]