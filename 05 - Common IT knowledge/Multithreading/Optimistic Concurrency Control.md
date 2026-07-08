---
aliases:
  - Optimistic Locking
---
Optimistic strategies involve taking a copy of the data, modifying it, then copying back the changes if data has not mutated in the meantime.  If a change has happened in the meantime you repeat the process until successful.  This repeating of the process increases with contention and therefore causes a queuing effect just like with [[Mutual Exclusion]].  

If you work with a source code control system, such as Subversion or CVS, then you are using this algorithm every day. Optimistic strategies can work with data but do not work so well with resources such as hardware because you cannot take a copy of the hardware!  The ability to perform the changes atomically to data is made possible by [[Compare-And-Swap|CAS]] instructions offered by the hardware.

Most locking strategies are composed from optimistic strategies for changing the lock state or mutual exclusion primitive. It means that **to build a "Pessimistic" lock, you actually have to use an "Optimistic" technique underneath.**