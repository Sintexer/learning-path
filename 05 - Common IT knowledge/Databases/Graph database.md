---
flashcards:
  q-nj4x: { nid: 1795356728592, hash: hqnkchfw, sync: d6y7xgri }
  q-h654: { nid: 1795356728617, hash: atubess6, sync: tyxf2hfs }
  q-uesk: { nid: 1795356728642, hash: repu9crk, sync: etuys2wc }
---

A graph consists of two kinds of objects: **vertices** (also known as nodes or entities) and **edges** (also known as relationships or arcs). Many kinds of data can be modeled as a graph. Typical examples include:
- **Social graphs**: Vertices are people, and edges indicate which people know each other.
- **The web graph**: Vertices are web pages, and edges indicate HTML links to other pages.
- **Road or rail networks**: Vertices are junctions, and edges represent the roads or railway lines between them.

Well-known algorithms can operate on these graphs: for example, car navigation systems search for the shortest path between two points in a road network, and PageRank can be used on the web graph to determine the popularity of a web page and thus its ranking in search results.
In the examples just given, all the vertices in a graph represent the same kind of thing (people, web pages, or road junctions, respectively). 

However, graphs are not limited to such homogeneous data: an equally powerful use of graphs is to provide a consistent way of storing completely different types of objects in a single datastore. For example, Facebook maintains a single graph with many different types of vertices and edges: vertices represent people, locations, events, check-ins, and comments made by users; edges indicate which people are friends with each other, which check-in happened in which location, who commented on which post, who attended which event, and so on.

There are several different, but related, ways of structuring and querying data in graphs: the **property graph** model (implemented by Neo4j, Titan, and InfiniteGraph) and the **triple-store model** (implemented by Datomic, AllegroGraph, and others).

## Property Graph

In the property graph model, each vertex consists of:
- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs) 

Each edge consists of:
- A unique identifier 
- The vertex at which the edge starts (the tail vertex)
- The vertex at which the edge ends (the head vertex)
- A label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)

The graph could be implemented using SQL using many-to-many relations up to some extent. For example graph traversal is painful in SQL, and easy in graph query languages.
### The Cypher Query Language

Cypher is a declarative query language for property graphs, created for the Neo4j graph database.

Each vertex is given a symbolic name like USA or Idaho, and other parts of the query can use those names to create edges between the vertices, using an arrow notation: `(Idaho) -[:WITHIN]-> (USA)` creates an edge labeled WITHIN, with Idaho as the tail node and USA as the head node.

## Triple-Stores

The triple-store model is mostly equivalent to the property graph model, using different words to describe the same ideas.

In a triple-store, all information is stored in the form of very simple three-part statements: (*subject*, *predicate*, *object*). For example, in the triple (*Jim*, *likes*, *bananas*), Jim is the subject, likes is the predicate (verb), and bananas is the object.

The subject of a triple is equivalent to a vertex in a graph. The object is one of two things:
1. A value in a primitive datatype, such as a string or a number. In that case, the predicate and object of the triple are equivalent to the key and value of a property on the subject vertex. For example, (lucy, age, 33) is like a vertex lucy with properties {"age":33}.
2. Another vertex in the graph. In that case, the predicate is an edge in the graph, the subject is the tail vertex, and the object is the head vertex. For example, in (lucy, marriedTo, alain) the subject and object lucy and alain are both vertices, and the predicate marriedTo is the label of the edge that connects them.

## SPARQL query language

SPARQL is a query language for triple-stores using the RDF data model. (It is an acronym for SPARQL Protocol and RDF Query Language, pronounced “sparkle.”) It predates Cypher, and since Cypher’s pattern matching is borrowed from SPARQL, they look quite similar.

Because RDF doesn’t distinguish between properties and edges but just uses predicates for both, you can use the same syntax for matching properties.

### RDF

The Resource Description Framework (RDF) was intended as a mechanism for different web‐
sites to publish data in a consistent format, allowing data from different websites to be automatically combined into a web of data—a kind of internet-wide “database of everything.”

RDF has a few quirks due to the fact that it is designed for internet-wide data exchange. The subject, predicate, and object of a triple are often URIs. For example, a predicate might be an URI such as `<http://my-company.com/namespace#within>` or `<http://my-company.com/namespace#lives_in>`, rather than just WITHIN or LIVES_IN. The reasoning behind this design is that you should be able to combine your data with someone else’s data, and if they attach a different meaning to the word within or lives_in, you won’t get a conflict because their predicates are actually `<http://other.org/foo#within>` and `<http://other.org/foo#lives_in>`.

## Datalog

Datalog is a much older language than SPARQL or Cypher, having been studied extensively by academics in the 1980s. It is less well known among software engineers, but it is nevertheless important, because it provides the foundation that later query languages build upon.
In practice, Datalog is used in a few data systems: for example, it is the query language of Datomic, and Cascalog is a Datalog implementation for querying large datasets in Hadoop.

> [!warning] Failure modes & trade-offs
> - **Supernodes (high-degree vertices):** A celebrity with 50 million followers creates a node with millions of edges; traversing through a supernode blows up latency and memory unless you cap degree, sample, or materialize edge indexes.
> - **Hard to partition horizontally:** Unlike key-value or document stores, graphs resist clean [[Data partitioning|sharding]] because multi-hop traversals cross partition boundaries and incur expensive network hops.
> - **When SQL is enough:** For shallow fixed-depth relationships (1–2 hops), a [[Relational Database]] with `WITH RECURSIVE` CTEs and proper [[Database Index|indexes]] is often simpler to operate than introducing a dedicated graph engine.

## Flashcards

When does a [[Graph database]] strongly beat both [[Relational Database|relational]] and [[Document Database|document]] databases?::When data is dominated by complex many-to-many relationships and variable-depth multi-hop traversals (e.g., fraud rings, social networks, knowledge graphs, permission hierarchies).
^q-nj4x

What are the two main graph data models?::The **Property Graph** model (vertices and labeled edges with key-value properties, queried with Cypher) and the **Triple-Store** model (`(subject, predicate, object)` statements, queried with SPARQL).
^q-h654

Why are [[Graph database|graph databases]] notoriously difficult to shard horizontally?::Multi-hop traversals follow arbitrary edges that frequently cross shard boundaries, turning fast in-memory pointer hops into high-latency distributed network round-trips.
^q-uesk

## Related

- [[NoSQL database]]
- [[Document Database]]
- [[Relational Database]]
- [[Query Languages for Data]]
- [[Data partitioning]]