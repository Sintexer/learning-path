---
flashcards:
  q-hxf7: { nid: 1795261497363, hash: jjxqfe3s, sync: hkfg22cy }
  q-w32s: { nid: 1795261497420, hash: 2vx2i4ey, sync: 3rr3ej2r }
---

## Happens-before Relation

An operation A happens before another operation B if B knows about A, or depends on A, or builds upon A in some way. Whether one operation **happens before** another operation is the key to defining what **concurrency** means. In fact, we can simply say that two operations are concurrent if neither happens before the other (i.e., neither knows about the other)

> [!tip] Key fact
> Whenever you have two operations A and B, there are three possibilities: either A happened before B, or B happened before A, or A and B are concurrent.

## Causally Dependent Write

If a write is like an increment, reading a value A and then applying a modification and writing a value B, in other word one write *happens-before* another write, we say that ::: the write B is **causally dependent** on write A.
^q-hxf7

## Concurrent Write

If writes A and B are happening at the same time and none of them depend on each others values, in other words there are no *happens-before relation*, such ::: writes are concurrent
^q-w32s

It may seem that two operations should be called concurrent if they occur “at the same time” —but in fact, *it is not important whether they literally overlap in time*. Because of problems with clocks in distributed systems, it is actually quite difficult to tell whether two things happened at exactly the same time.

For defining concurrency, exact time doesn’t matter: we simply call two operations concurrent if they are both **unaware of each other**, regardless of the physical time at which they occurred. 

In computer systems, two operations might be concurrent even though the speed of light would in principle have allowed one operation to affect the other. For example, if the network was slow or interrupted at the time, two operations can occur some time apart and still be concurrent, because the network problems prevented one operation from being able to know about the other.

