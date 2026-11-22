---
flashcards:
  q-vx4y: { nid: 1795356728920, hash: 5hav29z9, sync: hqwsngsp }
---

A database is an organized collection of data that is stored and accessed electronically. It allows you to store, manage, and retrieve data efficiently. Databases are used in a wide variety of applications, including:

* **Customer relationship management (CRM)**
* **E-commerce**
* **Financial transactions**
* **Inventory management**
* **Social media**

### Types of Databases:

There are many different types of databases, each with its own strengths and weaknesses. Some common types of databases include:

* **[[Relational Database]]** - These databases store data in tables with rows and columns. They are well-suited for structured data that needs to be accessed and manipulated in a specific way.
* **[[NoSQL database]]** - These databases store data in a more flexible format, such as JSON documents ([[Document Database]]), key-value pairs, wide columns ([[Apache Cassandra]]), or graphs ([[Graph database]]). They are well-suited for unstructured data or workloads needing elastic [[Horizontal scaling]].
* **[[In-Memory Database]]** - Keeps the primary dataset in RAM for sub-millisecond latency (e.g., [[Redis]], [[Memcached]]).
* **[[Cloud database]]** - These databases are hosted on a cloud platform, such as AWS or Azure. They offer scalability and flexibility, and they can be accessed from anywhere in the world.

### Database Management Systems ([[DBMS]])

A database management system ([[DBMS]]) is a software application that allows you to create, manage, and access databases. Some popular DBMSs include:

* **MySQL**
* **[[PostgreSQL]]**
* **Oracle Database**
* **Microsoft SQL Server**

### Benefits of Using a Database:

* **Data integrity:** Databases ensure that data is accurate and consistent (via constraints and [[ACID]] [[Transaction|transactions]]).
* **Data security:** Databases provide security measures to protect data from unauthorized access.
* **Data efficiency:** Databases allow you to store and retrieve data efficiently via [[Database Index|indexes]] and optimized [[Database Storage Engines|storage engines]].
* **Data analysis:** Databases can be used to analyze data and generate reports ([[OLAP]] / [[Data Warehouse]]).

> [!tip] Production reality
> Choosing a database is rarely about "which is fastest" — it is about matching the **data model** (relational vs. document vs. graph), **access pattern** ([[OLTP]] vs. [[OLAP]]), **consistency requirements** ([[ACID]] vs. [[Eventual Consistency]]), and **operational maturity** of the team. Default to a proven [[RDBMS]] like [[PostgreSQL]] unless a clear bottleneck demands a specialized store.

## Flashcards

What four dimensions should drive database selection in system design?::Data model (relationships), access pattern ([[OLTP]] vs [[OLAP]]), consistency/durability needs ([[ACID]] vs [[Eventual Consistency]]), and operational scalability/team maturity.
^q-vx4y

## Related

- [[DBMS]]
- [[RDBMS]]
- [[Relational Database]]
- [[NoSQL database]]
- [[SQL vs NoSQL]]
- [[Cloud database]]
- [[In-Memory Database]]
- [[Database Storage Engines]]
- [[Distributed Data]]