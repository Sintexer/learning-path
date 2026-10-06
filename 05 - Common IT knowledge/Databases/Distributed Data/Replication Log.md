This is how data is shared from [[Leader-Based Replication#Leader]] to [[Leader-Based Replication#Follower]]s.

## Statement-based replication
 
 The simplest case: the leader logs every write request (statement) that it executes and sends that statement log to its followers. Indirectly prohibits the usage of some statements like `RAND()`, `NOW()`, auto-incrementing columns, triggers and producers might also produce different results on different replicas.

## Write-ahead log (WAL) shipping

Default for [[PostgreSQL]]. DB shares the low-level [[WAL]] entries, with disk block identifiers and etc. Enforces hard requirement on [[Database Storage Engines]].  If the database changes its storage format from one version to another, it is typically not possible to run different versions of
the database software on the leader and the followers.

## Logical (row-based) log replication

An alternative is to use different log formats for replication and for the storage engine, which allows the replication log to be decoupled from the storage engine internals. This kind of replication log is called a logical log, to distinguish it from the storage engine’s (physical) data representation.

## Trigger-based replication

There are some circumstances where more flexibility is needed and the replication should be controlled from application code. For example, if you want to only replicate a subset of the data, or want to replicate from one kind of database to another, or if you need conflict resolution logic, then you may need to move replication up to the application layer.

Usually this solution is implemented using third-party tools like Oracle GoldenGate. It can make data changes available to an application by reading the database log. An alternative is to use features that are available in many relational databases: triggers and stored procedures.

A trigger lets you register custom application code that is automatically executed when a data change (write transaction) occurs in a database system. The trigger has the opportunity to log this change into a separate table, from which it can be read by an external process. That external process can then apply any necessary application logic and replicate the data change to another system. Databus for Oracle and Bucardo for Postgres work like this, for example.

Trigger-based replication typically has greater overheads than other replication methods, and is more prone to bugs and limitations than the database’s built-in replication. However, it can nevertheless be useful due to its flexibility.