Mutual exclusion is the means by which *only one writer can have access to a protected resource at a time*, and is usually implemented with a *locking strategy*. It is the way to implement the [[Pessimistic Concurrency Control]].

Locking strategies require an *arbitrator*, usually the operating system kernel, to get involved when the contention occurs to decide who gains access and in what order.  This can be a very expensive process often requiring many more CPU cycles than the actual transaction to be applied to the business logic would use.  

Those waiting to enter the [[Critical Section]] in advance of performing the mutation must queue, and this queuing effect ([[Little's Law]]) causes latency to become unpredictable and ultimately restricts throughput.