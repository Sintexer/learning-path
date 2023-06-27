## Publication

Thread-safe classes encapsulate any needed synchronization so that clients need not provide their own.

The most blatant form of publication is to store a reference in a public static field, where any class and thread could see it,

returning a reference from a nonprivate method also publishes the returned object.

Publishing an object also publishes any objects referred to by its nonprivate f i elds.More generally, any object that is reachable from a published object by following some chain of nonprivate f i eld references and method calls has also been published.

From the perspective of a class C, an alien method is one whose behavior is not fully specif i ed by C.This includes methods in other classes as well as overrideable methods (neitherprivatenorfinal) in C itself. Passing an object to an alien method must also be considered publishing that object. Since you can’t know what code will actually be invoked,

A final mechanism by which an object or its internal state can be published is to publish an inner class instance,

If data is only accessed from a single thread, no synchronization is needed. This technique, thread conf i nement,

An object is immutable if:
•Its state cannot be modified after construction;
•All its f i elds are final and 
•It is properly constructed (the this reference does not escape during construction).

*****

While it may seem that field values set in a constructor are the first values written to those fields and therefore that there are no “older” values to see as stale values, theObjectconstructor f i rst writes the default values to all f i elds before subclass constructors run. It is therefore possible to see the default value for a f i eld as a stale value.

To publish an object safely, both the reference to the object and the ob-ject’s state must be made visible to other threads at the same time. A properly constructed object can be safely published by:
•Initializing an object reference from a static initializer;
•Storing a reference to it into a volatile f i eld or AtomicReference;
•Storing a reference to it into a final f i eld of a properly constructed object; or •Storing a reference to it into a f i eld that is properly guarded by a lock.

The thread-safe library collections offer the following safe publication guarantees, even if the Javadoc is less than clear on the subject:
• Placing a key or value in aHashtable,synchronizedMap, orConcurrent-Mapsafely publishes it to any thread that retrieves it from theMap(whether directly or via an iterator);
• Placing an element in aVector,CopyOnWriteArrayList,CopyOnWrite-ArraySet,synchronizedList, orsynchronizedSetsafely publishes it to any thread that retrieves it from the collection;
• Placing an element on aBlockingQueueor aConcurrentLinkedQueuesafely publishes it to any thread that retrieves it from the queue.

Using a static initializer is often the easiest and safest way to publish objects that can be statically constructed:
public static Holder holder = new Holder(42);
Static initializers are executed by the JVM at class initialization time; because of internal synchronization in the JVM, this mechanism is guaranteed to safely publish any objects initialized in this way [JLS 12.4.2].

Objects that are not technically immutable, but whose state will not be mod-if i ed after publication, are called effectively immutable.

Safely published effectively immutable objects can be used safely by any thread without additional synchronization.

*****

The publication requirements for an object depend on its mutability:
•Immutable objects can be published through any mechanism;
•Ef f ectively immutable objects must be safely published;
•Mutable objects must be safely published, and must be either thread-safe or guarded by a lock.

*****

The most useful policies for using and sharing objects in a concurrent program are:
Thread-conf i ned. A thread-conf i ned object is owned exclusively by and conf i ned to one thre

Ilya, [6/27/2023 12:25 PM]
ad, and can be modif i ed by its owning thread.
Shared read-only. A shared read-only object can be accessed concur-rently by multiple threads without additional synchronization, but cannot be modif i ed by any thread. Shared read-only objects include immutable and ef f ectively immutable objects.
Shared thread-safe. A thread-safe object performs synchronization in-ternally, so multiple threads can freely access it through its public interface without further synchronization.
Guarded. A guarded object can be accessed only with a specif i c lock held. Guarded objects include those that are encapsulated within other thread-safe objects and published objects that are known to be guarded by a specif i c lock.

*****

The design process for a thread-safe class should include these three basic elements:
•Identify the variables that form the object’s state;
•Identify the invariants that constrain the state variables;
•Establish a policy for managing concurrent access to the object’s state.

*****

By using f i nal f i elds wherever practical, you make it simpler to analyze the possible states an object can be in. (In the extreme case, immutable objects can only be in a single state.) Many classes have invariants that identify certain states as valid or invalid.
Thevaluef i eld inCounteris along.The state space of alongranges from Long.MIN_VALUEtoLong.MAX_VALUE, butCounterplaces constraints onvalue;
negative values are not allowed.
Similarly, operations may have postconditions that identify certain state transi-tions as invalid. If the current state of aCounteris 17, the only valid next state is 18. When the next state is derived from the current state, the operation is neces-sarily a compound action. Not all operations impose state transition constraints;
when updating a variable that holds the current temperature, its previous state does not affect the computation.

*****

Constraints placed on states or state transitions by invariants and postcondi-tions create additional synchronization or encapsulation requirements. If certain states are invalid, then the underlying state variables must be encapsulated, oth-erwise client code could put the object into an invalid state. If an operation has invalid state transitions, it must be made atomic.

*****

You cannot ensure thread safety without understanding an object’s invari-ants and postconditions. Constraints on the valid values or state transi-tions for state variables can create atomicity and encapsulation require-ments.

*****

Some objects also have methods with state-based precon-ditions. For example, you cannot remove an item from an empty queue; a queue must be in the “nonempty” state before you can remove an element. Operations with state-based preconditions are called state-dependent

*****

In many cases, ownership and encapsulation go together—the object encapsu-lates the state it owns and owns the state it encapsulates. It is the owner of a given state variable that gets to decide on the locking protocol used to maintain the in-tegrity of that variable’s state.

*****

the object encapsu-lates the state it owns and owns the state it encapsulates.

*****

Ownership implies control, but once you publish a reference to a mutable object, you no longer have exclusive control; at best, you might have “shared ownership”. A class usually does not own the objects passed to its methods or constructors, unless the method is designed to explicitly trans-fer ownership of objects passed in (such as the synchronized collection wrapper factory methods).

*****

Collection classes often exhibit a form of “split ownership”, in which the col-lection owns the state of the collection infrastructure, but client code owns the objects stored in the collection. An example isServletContextfrom the serv-let framework

*****

framework.ServletContextprovides aMap-like object container service to servlets where they can register and retrieve application objects by name with setAttributeandgetAttribute. TheServletContextobject implemented by the servlet container must be thread-safe, because it wi

Ilya, [6/27/2023 12:25 PM]
ll necessarily be accessed by multiple threads. Servlets need not use synchronization when callingset-AttributeandgetAttribute, but they may have to use synchronization when using the objects stored in theServletContext. These objects are owned by the application; they are being stored for safekeeping by the servlet container on the application’s behalf. Like all shared objects, they must be shared safely; in or-der to prevent interference from multiple threads accessing the same object con-currently, they should either be thread-safe, effectively immutable, or explicitly guarded by a lock.1

*****

Instance conf i nement

*****

it is only accessed from a single thread (thread conf i nement),

*****

Encapsulating data within an object conf i nes access to the data to the ob-ject’s methods, making it easier to ensure that the data is always accessed with the appropriate lock held.

*****

scope.An object may be conf i ned to a class instance (such as a private class member), a lexical scope (such as a local variable), or a thread (such as an object that is passed from method to method within a thread, but not supposed to be shared across threads). Objects don’t escape on their own, of course—they need help from the developer, who assists by publishing the object beyond its intended scope.

*****

Conf i nement makes it easier to build thread-safe classes because a class that conf i nes its state can be analyzed for thread safety without having to examine the whole program.

*****

There are advantages to using a private lock object instead of an object’s in-trinsic lock (or any other publicly accessible lock). Making the lock object private encapsulates the lock so that client code cannot acquire it, whereas a publicly accessible lock allows client code to participate in its synchronization policy— correctly or incorrectly. Clients that improperly acquire another object’s lock could cause liveness problems, and verifying that a publicly accessible lock is properly used requires examining the entire program rather than a single class.

*****

Delegating thread safety

*****

We could say thatCountingFactorizerdelegates its thread safety responsibilities to theAtom-icLong:CountingFactorizeris thread-safe becauseAtomicLongis

*****

Pointis thread-safe because it is immutable. Immutable values can be freely shared and published, so we no longer need to copy the locations when returning them.

*****

The delegation examples so far delegate to a single, thread-safe state variable. We can also delegate thread safety to more than one underlying state variable as long as those underlying state variables are independent, meaning that the composite class does not impose any invariants involving the multiple state variables.

*****

EachListis thread-safe, and because there are no con-straints coupling the state of one to the state of the other,VisualComponentcan delegate its thread safety responsibilities to the underlyingmouseListenersand keyListenersobjects.

*****

NumberRangeis not thread-safe; it does not preserve the invariant that con-strainslowerandupper. ThesetLowerandsetUppermethods attempt to respect this invariant, but do so poorly. BothsetLowerandsetUpperare check-then-act sequences, but they do not use suff i cient locking to make them atomic.

*****

If a class is composed of multiple independent thread-safe state variables and has no operations that have any invalid state transitions, then it can delegate thread safety to the underlying state variables.

*****

When you delegate thread safety to an object’s underlying state variables, under what conditions can you publish those variables so that other classes can modify them as well?

*****

If a state variable is thread-safe, does not participate in any invariants that constrain its value, and has no prohibited state transitions for any of its operations, then it can safely be published.

*****

4.4 Adding functionality to existing thread-safe classes

*****

The safest way to add a new atomic operation is to modify the original class to support the desired opera

Ilya, [6/27/2023 12:25 PM]
tion, but this is not always possible because you may not have access to the source code or may not be free to modify it. If you can modify the original class, you need to understand the implementation’s synchro-nization policy so that you can enhance it in a manner consistent with its original design. Adding the new method directly to the class means that all the code that implements the synchronization policy for that class is still contained in one source f i le, facilitating easier comprehension and maintenance.

*****

Another approach is to extend the class, assuming it was designed for exten-sion

*****

Extension is more fragile than adding code directly to a class, because the implementation of the synchronization policy is now distributed over multiple, separately maintained source f i les.If the underlying class were to change its synchronization policy by choosing a different lock to guard its state variables, the subclass would subtly and silently break, because it no longer used the right lock to control concurrent access to the base class’s state.

*****

For anArrayListwrapped with aCollections.synchronizedListwrapper, nei-ther of these approaches—adding a method to the original class or extending the class—works because the client code does not even know the class of theList object returned from the synchronized wrapper factories.

*****

A third strategy is to extend the functionality of the class without extending the class itself by placing extension code in a “helper” class.

*****

To make this approach work, we have to use the same lock that theListuses by using client-side locking or external locking. Client-side locking entails guarding client code that uses some object X with the lock X uses to guard its own state.
In order to use client-side locking, you must know what lock X uses.

*****

Document a class’s thread safety guarantees for its clients; document its synchronization policy for its maintainers.

*****

The synchronized collections are thread-safe, but you may sometimes need to use additional client-side locking to guard compound actions. Common compound actions on collections include iteration (repeatedly fetch elements until the collec-tion is exhausted), navigation (f i nd the next element after this one according to some order), and conditional operations such as put-if-absent (check if aMaphas a mapping for key K, and if not, add the mapping(K, V)). With a synchronized collection, these compound actions are still technically thread-safe even without client-side locking, but they may not behave as you might expect when other threads can concurrently modify the collection.

*****

If thread A callsgetLaston aVectorwith ten elements, thread B callsdeleteLaston the sameVector, and the operations are interleaved as shown in Figure 5.1,getLastthrowsArrayIn-dexOutOfBoundsException. Between the call tosizeand the subsequent call to getingetLast, theVectorshrank and the index computed in the f i rst step is no longer valid.

*****

Because the synchronized collections commit to a synchronization policy that supports client-side locking,1it is possible to create new operations that are atomic with respect to other collection operations as long as we know which lock to use.

*****

This iteration idiom relies on a leap of faith that other threads will not modify theVectorbetween the calls tosizeandget. In a single-threaded environment, this assumption is perfectly valid, but when other threads may concurrently mod-ify theVectorit can lead to trouble.Just as withgetLast, if another thread deletes an element while you are iterating through theVectorand the operations are interleaved unluckily, this iteration idiom throwsArrayIndexOutOfBoundsEx-ception.

*****

Even though the iteration in Listing 5.3 can throw an exception, this doesn’t meanVectorisn’t thread-safe. The state of theVectoris still valid and the ex-ception is in fact in conformance with its specif i cation. However, that something as mundane as fetching the last element or iteration throw an exception is clearly undesirable.

**

Ilya, [6/27/2023 12:25 PM]
***

ConcurrentModificationExceptioncan arise in single-threaded code as well; this happens when objects are removed from the collection directly rather than throughIterator.remove.

*****

Also, if the collection is locked as in Listing 5.4,doSomethingis being called with a lock held, which is a risk factor for deadlock

*****

Just as encapsulating an object’s state makes it easier to preserve its in-variants, encapsulating its synchronization makes it easier to enforce its synchronization policy.

*****

Iteration is also indirectly invoked by the collection’shashCodeandequals methods, which may be called if the collection is used as an element or key of another collection. Similarly, thecontainsAll,removeAll, andretainAllmeth-ods, as well as the constructors that take collections as arguments, also iterate the collection. All of these indirect uses of iteration can causeConcurrentModifica-tionException.

*****

Replacing synchronized collections with concurrent collections can of f er dramatic scalability improvements with little risk.

*****

implementsQueue—theQueueclasses were added because eliminating the random-access requirements ofListadmits more eff i cient con-current implementations.

*****

BlockingQueueextendsQueueto add blocking insertion and retrieval opera-tions. If the queue is empty, a retrieval blocks until an element is available, and if the queue is full (for bounded queues) an insertion blocks until there is space available. Blocking queues are extremely useful in producer-consumer designs, and are covered in greater detail in Section 5.3.

*****

Instead of synchronizing every method on a common lock, restricting access to a single thread at a time, it uses a f i ner-grained locking mechanism called lock striping (see Section 11.4.3) to allow a greater degree of shared access. Arbitrarily many reading threads can access the map concurrently, readers can access the map concurrently with writers, and a limited number of writers can modify the map concurrently. The result is far higher throughput under concurrent access, with little performance penalty for single-threaded access.

*****

ConcurrentHashMap, along with the other concurrent collections, further im-prove on the synchronized collection classes by providing iterators that do not throwConcurrentModificationException, thus eliminating the need to lock the collection during iteration. The iterators returned byConcurrentHashMapare weakly consistent instead of fail-fast. A weakly consistent iterator can tolerate con-current modif i cation, traverses elements as they existed when the iterator was constructed, and may (but is not guaranteed to) ref l ect modif i cations to the col-lection after the construction of the iterator.

*****

collection.Since the result ofsizecould be out of date by the time it is computed, it is really only an estimate, sosizeis allowed to return an approximation instead of an exact count. While at f i rst this may seem disturbing, in reality methods likesizeand isEmptyare far less useful in concurrent environments because these quantities are moving targets. So the requirements for these operations were weakened to enable performance optimizations for the most important operations, primarily get,put,containsKey, andremove.

*****

Because it has so many advantages and so few disadvantages compared to HashtableorsynchronizedMap, replacing synchronizedMapimplementations withConcurrentHashMapin most cases results only in better scalability. Only if your application needs to lock the map for exclusive access3isConcurrent-HashMapnot an appropriate drop-in replacement.

*****

5.2.2 Additional atomicMapoperations Since aConcurrentHashMapcannot be locked for exclusive access, we cannot use client-side locking to create new atomic operations such as put-if-absent, as we did forVectorin Section 4.4.1. Instead, a number of common compound opera-tions such as put-if-absent, remove-if-equal, and replace-if-equal are implemented as atomic operations and specif i ed by theConcurrentMapinterface, shown in List-ing 5.7. If you f i

Ilya, [6/27/2023 12:25 PM]
nd yourself adding such functionality to an existing synchronized Mapimplementation, it is probably a sign that you should consider using aCon-currentMapinstead.

*****

The copy-on-write collections derive their thread safety from the fact that as long as an effectively immutable object is properly published, no further synchro-nization is required when accessing it.

*****

As a result, multiple threads can iterate the collection without interference from one another or from threads wanting to modify the collection. The iterators returned by the copy-on-write collections do not throwConcurrentModificationExceptionand return the elements exactly as they were at the time the iterator was created, regardless of subsequent modif i cations.

*****

the copy-on-write col-lections are reasonable to use only when iteration is far more common than mod-if i cation. This criterion exactly describes many event-notif i cation systems: deliv-ering a notif i cation requires iterating the list of registered listeners and calling each one of them, and in most cases registering or unregistering an event listener is far less common than receiving an event notif i cation. (See [CPJ 2.4.4] for more information on copy-on-write.)

*****

Blocking queues provide blockingputandtakemethods as well as the timed equivalentsofferandpoll. If the queue is full,putblocks until space becomes available; if the queue is empty,takeblocks until an element is available. Queues can be bounded or unbounded; unbounded queues are never full, so aputon an unbounded queue never blocks.

*****

Blocking queues also provide anoffermethod, which returns a failure status if the item cannot be enqueued. This enables you to create more f l exible policies for dealing with overload, such as shedding load, serializing excess work items and writing them to disk, reducing the number of producer threads, or throttling producers in some other manner.

*****

Bounded queues are a powerful resource management tool for building reliable applications: they make your program more robust to overload by throttling activities that threaten to produce more work than can be handled.

*****

Build resource management into your design early using blocking queues—it is a lot easier to do this up front than to retrof i t it later. Blocking queues make this easy for a number of situations, but if blocking queues don’t f i t easily into your design, you can create other blocking data structures usingSemaphore(see Section 5.5.3).

*****

natural order (if they implementComparable) or using aComparator.
The lastBlockingQueueimplementation,SynchronousQueue, is not really a queue at all, in that it maintains no storage space for queued elements. Instead, it maintains a list of queued threads waiting to enqueue or dequeue an element.
In the dish-washing analogy, this would be like having no dish rack, but instead handing the washed dishes directly to the next available dryer. While this may seem a strange way to implement a queue, it reduces the latency associated with moving data from producer to consumer because the work can be handed off directly. (In a traditional queue, the enqueue and dequeue operations must com-plete sequentially before a unit of work can be handed off.) The direct handoff also feeds back more information about the state of the task to the producer;
when the handoff is accepted, it knows a consumer has taken responsibility for it,

*****

Java 6 also adds another two collection types,Deque(pronounced “deck”) and BlockingDeque, that extendQueueandBlockingQueue. ADequeis a double-ended queue that allows eff i cient insertion and removal from both the head and the tail. Implementations includeArrayDequeandLinkedBlockingDeque.

*****

Just as blocking queues lend themselves to the producer-consumer pattern, deques lend themselves to a related pattern called work stealing.A producer-consumer design has one shared work queue for all consumers; in a work stealing design, every consumer has its own deque. If a consumer exhausts the work in its own deque, it can steal work from

Ilya, [6/27/2023 12:25 PM]
the tail of someone else’s deque.

*****

Work stealing is well suited to problems in which consumers are also produc-ers—when performing a unit of work is likely to result in the identif i cation of more work. For example, processing a page in a web crawler usually results in the identif i cation of new pages to be crawled. Similarly, many graph-exploring algorithms, such as marking the heap during garbage collection, can be eff i ciently parallelized using work stealing. When a worker identif i es a new unit of work, it places it at the end of its own deque (or alternatively, in a work sharing design, on that of another worker); when its deque is empty, it looks for work at the end of someone else’s deque, ensuring that each worker stays busy.

*****

When your code calls a method that throwsInterruptedException, then your method is a blocking method too, and must have a plan for responding to inter-ruption. For library code, there are basically two choices:
Propagate theInterruptedException. This is often the most sensible policy if you can get away with it—just propagate theInterruptedExceptionto your caller. This could involve not catchingInterruptedException, or catching it and throwing it again after performing some brief activity-specif i c cleanup.
Restore the interrupt. Sometimes you cannot throwInterruptedException, for instance when your code is part of aRunnable. In these situations, you must catchInterruptedExceptionand restore the interrupted status by calling interrupton the current thread, so that code higher up the call stack can see that an interrupt was issued, as demonstrated in Listing 5.10.

*****

But there is one thing you should not do withInterruptedException—catch it and do nothing in re-sponse. This deprives code higher up on the call stack of the opportunity to act on the interruption, because the evidence that the thread was interrupted is lost. The only situation in which it is acceptable to swallow an interrupt is when you are extending Threadand therefore control all the code higher up on the call stack. Cancellation and interruption are covered in greater detail in Chapter 7.

*****

Blocking queues are unique among the collections classes: not only do they act as containers for objects, but they can also coordinate the control f l ow of producer and consumer threads becausetakeandputblock until the queue enters the desired state (not empty or not full).

*****

5.5.1 Latches

*****

A latch is a synchronizer that can delay the progress of threads until it reaches its terminal state [CPJ 3.4.2]. A latch acts as a gate: until the latch reaches the terminal state the gate is closed and no thread can pass, and in the terminal state the gate opens, allowing all threads to pass. Once the latch reaches the terminal state, it cannot change state again, so it remains open forever.

*****

Latches can be used to ensure that certain activities do not proceed until other one-time activities complete, such as:
• Ensuring that a computation does not proceed until resources it needs have been initialized. A simple binary (two-state) latch could be used to indicate “Resource R has been initialized”, and any activity that requires R would wait f i rst on this latch.

*****

Ensuring that a service does not start until other services on which it de-pends have started. Each service would have an associated binary latch;
starting service S would involve f i rst waiting on the latches for other ser-vices on which S depends, and then releasing the S latch after startup com-pletes so any services that depend on S can then proceed.
• Waiting until all the parties involved in an activity, for instance the players in a multi-player game, are ready to proceed. In this case, the latch reaches the terminal state after all the players are ready.

*****

FutureTask

*****

FutureTaskalso acts like a latch. (FutureTaskimplementsFuture, which de-scribes an abstract result-bearing computation [CPJ 4.3.3].) A computation rep-resented by aFutureTaskis implemented with aCallable, the result-bearing equivalent ofRunnable,

Ilya, [6/27/2023 12:25 PM]
and can be in one of three states: waiting to run, running, or completed. Completion subsumes all the ways a computation can complete, including normal completion, cancellation, and exception. Once aFutureTask enters the completed state, it stays in that state forever.

*****

Tasks described byCallablecan throw checked and unchecked exceptions, and any code can throw anError

*****

wrapped in anExecutionExceptionand rethrown fromFuture.get. This com-plicates code that callsget, not only because it must deal with the possibility of ExecutionException(and the uncheckedCancellationException), but also be-cause the cause of theExecutionExceptionis returned as aThrowable, which is inconvenient to deal with.

*****

5.5.3 Semaphores Counting semaphores are used to control the number of activities that can access a certain resource or perform a given action at the same time [CPJ 3.4.1]. Counting semaphores can be used to implement resource pools or to impose a bound on a collection.

*****

Semaphores are useful for implementing resource pools such as database con-nection pools. While it is easy to construct a f i xed-sized pool that fails if you request a resource from an empty pool, what you really want is to block if the pool is empty and unblock when it becomes nonempty again. If you initialize aSemaphoreto the pool size,acquirea permit before trying to fetch a resource from the pool, andreleasethe permit after putting a resource back in the pool, acquireblocks until the pool becomes nonempty. This technique is used in the bounded buffer class in Chapter 12. (An easier way to construct a blocking object pool would be to use aBlockingQueueto hold the pooled resources.)

*****

Similarly, you can use aSemaphoreto turn any collection into a block-ing bounded collection, as illustrated byBoundedHashSetin Listing 5.14. The semaphore is initialized to the desired maximum size of the collection. Theadd operation acquires a permit before adding the item into the underlying collec-tion. If the underlyingaddoperation does not actually add anything, it releases the permit immediately. Similarly, a successfulremoveoperation releases a per-mit, enabling more elements to be added. The underlyingSetimplementation knows nothing about the bound; this is handled byBoundedHashSet.

*****

5.5.4 Barriers We

*****

We have seen how latches can facilitate starting a group of related activities or waiting for a group of related activities to complete. Latches are single-use objects;
once a latch enters the terminal state, it cannot be reset.

*****

CyclicBarrierallows a f i xed number of parties to rendezvous repeatedly at a barrier point and is useful in parallel iterative algorithms that break down a problem into a f i xed number of independent subproblems. Threads callawait when they reach the barrier point, andawaitblocks until all the threads have reached the barrier point. If all threads meet at the barrier point, the barrier has been successfully passed, in which case all threads are released and the barrier is reset so it can be used again. If a call toawaittimes out or a thread blocked in awaitis interrupted, then the barrier is considered broken and all outstanding calls toawaitterminate withBrokenBarrierException.

*****

Barriers are often used in simulations, where the work to calculate one step can be done in parallel but all the work associated with a given step must complete before advancing to the next step. For example, in n-body particle simulations, each step calculates an update to the position of each particle based on the lo-cations and other attributes of the other particles. Waiting on a barrier between each update ensures that all updates for step k have completed before moving on to step k+1.

*****

Another form of barrier isExchanger, a two-party barrier in which the parties exchange data at the barrier point [CPJ 3.4.3]. Exchangers are useful when the parties perform asymmetric activities, for example when one thread f i lls a buffer with data and the other thread consumes the data from the buffer; these threads

Ilya, [6/27/2023 12:25 PM]
could use anExchangerto meet and exchange a full buffer for an empty one.
When two threads exchange objects via anExchanger, the exchange constitutes a safe publication of both objects to the other party.

*****

Like many other frequently reinvented wheels, caching often looks simpler than it is. A naive cache implementation is likely to turn a performance bottleneck into a scalability bottleneck, even if it does improve single-threaded performance.
In this section we develop an eff i cient and scalable result cache for a compu-tationally expensive function. Let’s start with the obvious approach—a simple HashMap—and then look at some of its concurrency disadvantages and how to f i x them.

*****

Ideally, tasks are independent activities: work that doesn’t depend on the state, result, or side effects of other tasks.

*****

Thread lifecycle overhead. Thread creation and teardown are not free. The ac-tual overhead varies across platforms, but thread creation takes time, intro-ducing latency into request processing, and requires some processing activ-ity by the JVM and OS. If requests are frequent and lightweight, as in most server applications, creating a new thread for each request can consume signif i cant computing resources.

*****

TheExecutorframework Tasks

*****

Thread pools offer the same benef i t for thread management, andjava.util.concurrentprovides a f l exible thread pool implementation as part of theExecutorframework.

*****

newFixedThreadPool. A f i xed-size thread pool creates threads as tasks are sub-mitted, up to the maximum pool size, and then attempts to keep the pool size constant (adding new threads if a thread dies due to an unexpected Exception).
newCachedThreadPool. A cached thread pool has more f l exibility to reap idle threads when the current size of the pool exceeds the demand for process-ing, and to add new threads when demand increases, but places no bounds on the size of the pool.
newSingleThreadExecutor. A single-threaded executor creates a single worker thread to process tasks, replacing it if it dies unexpectedly. Tasks are guar-anteed to be processed sequentially according to the order imposed by the task queue (FIFO, LIFO, priority order).4 newScheduledThreadPool. A f i xed-size thread pool that supports delayed and periodic task execution, similar toTimer. (See Section 6.2.5.)

*****

Because anExecutorprocesses tasks asynchronously, at any given time the state of previously submitted tasks is not immediately obvious. Some may have completed, some may be currently running, and others may be queued awaiting execution. In shutting down an application, there is a spectrum from graceful shutdown (f i nish what you’ve started but don’t accept any new work) to abrupt shutdown (turn off the power to the machine room), and various points in be-tween

*****

TheTimerfacility manages the execution of deferred (“run this task in 100 ms”) and periodic (“run this task every 10 ms”) tasks. However,Timerhas some draw-backs, andScheduledThreadPoolExecutorshould be thought of as its replace-ment.

*****

ATimercreates only a single thread for executing timer tasks.If a timer task takes too long to run, the timing accuracy of otherTimerTasks can suffer.
If a recurringTimerTaskis scheduled to run every 10 ms and anotherTimer-Tasktakes 40 ms to run, the recurring task either (depending on whether it was scheduled at f i xed rate or f i xed delay) gets called four times in rapid succession after the long-running task completes, or “misses” four invocations completely.
Scheduled thread pools address this limitation by letting you provide multiple threads for executing deferred and periodic tasks.

*****

Another problem withTimeris that it behaves poorly if aTimerTaskthrows an unchecked exception. TheTimerthread doesn’t catch the exception, so an un-checked exception thrown from aTimerTaskterminates the timer thread.Timer also doesn’t resurrect the thread in this situation; instead, it erroneously assumes the entireTimerwas cancelled.

*****

If you need to build your own scheduling service, you may s

Ilya, [6/27/2023 12:25 PM]
till be able to take advantage of the library by using aDelayQueue, aBlockingQueueimplementation that provides the scheduling functionality ofScheduledThreadPoolExecutor. A DelayQueuemanages a collection ofDelayedobjects. ADelayedhas a delay time associated with it:DelayQueuelets youtakean element only if its delay has expired. Objects are returned from aDelayQueueordered by the time associated with their delay.

*****

Result-bearing tasks:CallableandFuture

*****

tasks,Callableis a better abstraction: it expects that the main entry point,call, will return a value and anticipates that it might throw an exception

*****

In theExecutorframework, tasks that have been submitted but not yet started can always be cancelled, and tasks that have started can some-times be cancelled if they are responsive to interruption. Cancelling a task that has already completed has no effect.

*****

Futurerepresents the lifecycle of a task and provides methods to test whether the task has completed or been cancelled, retrieve its result, and cancel the task.

*****

The behavior ofgetvaries depending on the task state (not yet started, run-ning, completed). It returns immediately or throws anExceptionif the task has already completed, but if not it blocks until the task completes. If the task completes by throwing an exception,getrethrows it wrapped in anExecution

*****

There are several ways to create aFutureto describe a task. Thesubmit methods inExecutorServiceall return aFuture, so that you can submit aRunn-ableor aCallableto an executor and get back aFuturethat can be used to retrieve the result or cancel the task. You can also explicitly instantiate aFut-ureTaskfor a givenRunnableorCallable.(BecauseFutureTaskimplements Runnable, it can be submitted to anExecutorfor execution or executed directly by calling itsrunmethod.)

*****

CompletionServicecombines the functionality of anExecutorand aBlock-ingQueue. You can submitCallabletasks to it for execution and use the queue-like methodstakeandpollto retrieve completed results, packaged asFutures, as they become available.ExecutorCompletionServiceimplementsCompletion-Service, delegating the computation to anExecutor.

*****

MultipleExecutorCompletionServices can share a singleExecutor,

*****

create anExecutorCompletionServicethat is private to a particular computation while sharing a commonExecutor. When used in this way, aCompletionServiceacts as a handle for a batch of computations in much the same way that aFutureacts as a handle for a single computation.

*****

By remem-bering how many tasks were submitted to theCompletionServiceand counting how many completed results are retrieved, you can know when all the results for a given batch have been retrieved, even if you use a sharedExecutor.

*****

The primary challenge in executing tasks within a time budget is making sure that you don’t wait longer than the time budget to get an answer or f i nd out that one is not forthcoming. The timed version ofFuture.getsupports this requirement: it returns as soon as the result is ready, but throwsTimeoutExcep-tionif the result is not ready within the timeout period.

*****

if a timedgetcompletes with aTimeoutException, you can cancel the task through theFuture. If the task is written to be cancellable (see Chapter 7), it can be terminated early so as not to consume excessive resources.

*****

Listing 6.17 uses the timed version ofinvokeAllto submit multiple tasks to anExecutorServiceand retrieve the results.TheinvokeAllmethod takes a collection of tasks and returns a collection ofFutures. The two collections have identical structures;invokeAlladds theFutures to the returned collection in the order imposed by the task collection’s iterator, thus allowing the caller to associate aFuturewith theCallableit represents. The timed version ofinvokeAllwill return when all the tasks have completed, the calling thread is interrupted, or the timeout expires. Any tasks that are not complete when the timeout expires are cancelled. On return frominvokeAll, each task will have either completed normally

Ilya, [6/27/2023 12:25 PM]
or been cancelled; the client code can callgetorisCancelled

*****

Summary Structuring applications around the execution of tasks can simplify development and facilitate concurrency. TheExecutorframework permits you to decouple task submission from execution policy and supports a rich variety of execution policies; whenever you f i nd yourself creating threads to perform tasks, consider using anExecutorinstead. To maximize the benef i t of decomposing an applica-tion into tasks, you must identify sensible task boundaries. In some applications, the obvious task boundaries work well, whereas in others some analysis may be required to uncover f i ner-grained exploitable parallelism.

--
Reading books with ReadEra
https://play.google.com/store/apps/details?id=org.readera&hl=en
