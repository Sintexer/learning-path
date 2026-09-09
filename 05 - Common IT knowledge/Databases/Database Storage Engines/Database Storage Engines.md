
Database engines mainly diverged in two directions:
- [[OLTP]] oriented
- [[OLAP]] oriented

## Simplest database case 

The world's simplest database could be implemented as two bash functions: one write a key-value pair to a file, and the other reads the first match for a key. Such database has an incredibly good performance for write. It is effectively an easiest and most naive implementation of the [[Database Log]]. However it also has a bad read performance as it has to iterate $O(N)$ to find the matching record.

That is why real databases need an [[Database Index|index]] - a data structure for finding the value for a particular key efficiently. 