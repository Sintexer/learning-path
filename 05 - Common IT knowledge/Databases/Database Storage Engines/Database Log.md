Databases usually use *log*, which is usually different from the application logger. Database log is an append only file, that has a reasonable good performance. Databases usually also have to deal with concurrency control, reclaiming disk space so that the log doesn't grow forever, and handling errors and partially written records.

## Simplest case

A simplest example is a running append-only file for key-value data. Each new write is appended to the end of file. And to support quick data access, it might offer a [[Database Index#Hash Index|hash index]] using an in-memory hash map.
