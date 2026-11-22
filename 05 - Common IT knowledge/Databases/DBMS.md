---
flashcards:
  q-pe2k: { nid: 1795356728966, hash: vj9sicpb, sync: 66q7suib }
---

> AKA **Data Base Management System**

## DBMS Explained

A Database Management System (DBMS) is software that allows users to create, manage, and access [[Database|databases]]. It acts as an interface between the database and users, providing various functionalities like:

- **Data Definition:** Defining the structure of the database, including tables, columns, and relationships.
- **Data Manipulation:** Inserting, updating, deleting, and querying data within the database.
- **Data Security:** Ensuring data integrity, consistency, and access control through user authentication and authorization.
- **Data Concurrency:** Managing concurrent access to the database by multiple users, preventing data corruption.
- **Data Recovery:** Providing mechanisms to recover data in case of system failures or accidental deletion.
- **Performance Optimization:** Tuning and optimizing database queries for faster data retrieval and processing.
- **Backup and Recovery:** Backing up the database regularly and restoring it in case of failures.
- **User Management:** Creating and managing user accounts with different access levels and permissions.
- **Transaction Management:** Ensuring data consistency and integrity by grouping multiple operations into atomic units.
- **Reporting and Analysis:** Generating reports and analyzing data stored in the database.

There is a specific DBMS type for [[Relational Database|Relational Databases]] called [[RDBMS]].

**DBMS Examples:**

* MySQL
* [[PostgreSQL]]
* Oracle Database
* Microsoft SQL Server
* IBM Db2
* AWS Aurora

**Benefits of Using a DBMS:**

* Improved data integrity and consistency.
* Enhanced data security and access control.
* Efficient data management and retrieval.
* Scalability and flexibility to handle large datasets.
* Improved data analysis and reporting capabilities.

**Note:** DBMS is a crucial component for managing and utilizing data effectively in various applications, from small-scale personal databases to large-scale enterprise systems.

> [!tip] Production reality
> Internally, every modern DBMS splits cleanly into layered subsystems: **Connection/Query Layer** (parser, planner/optimizer, executor) → **Transaction & Concurrency Manager** ([[MVCC]], [[Database Lock|locks]], [[ACID]]) → **[[Database Storage Engines|Storage Engine]]** (buffer pool, [[B-Tree Index|B-Tree]]/[[LSM Tree]], [[WAL]]). Knowing which layer owns a bottleneck (e.g., bad planner stats vs. lock contention vs. buffer-pool misses) is how Staff engineers debug database incidents fast.

## Flashcards

What are the three core internal layers of a modern [[DBMS]]?::1) Query processing layer (parser, optimizer, executor), 2) Transaction & concurrency manager ([[MVCC]], [[Database Lock|locks]]), and 3) [[Database Storage Engines|Storage engine]] (buffer pool, indexes, [[WAL]]).
^q-pe2k

## Related

- [[Database]]
- [[RDBMS]]
- [[Database Storage Engines]]
- [[Transaction]]
- [[ACID]]
- [[PostgreSQL]]

