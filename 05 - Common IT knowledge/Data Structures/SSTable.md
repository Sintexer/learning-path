**SSTables (Sorted String Tables)** are the foundational disk-based building blocks of an [[LSM Tree]].

**Definition:** An SSTable is an immutable, sorted file of key-value pairs. Once an SSTable is written to disk, it **never changes**.

- An LSM-Tree is essentially a hierarchy and management strategy for multiple SSTables across different levels on disk, combined with an in-memory buffer ([[LSM Tree#Memtable]]).
- Every time a Memtable is flushed to disk, a **new SSTable is born**.
- Because data is continuously written, an LSM-Tree accumulates many individual SSTables over time.