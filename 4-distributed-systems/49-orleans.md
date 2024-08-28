# Orleans

Link: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/Orleans-MSR-TR-2014-41.pdf

Read June 30th, 2024.

# Introduction
* Building interactive services that are scalable and reliable is hard.
* Target workload: Interactive, **latency-sensistive**, run for a finite amounts of time
* Implementing virtual actors with a limited underlying number of physical actors
* Object-Oriented Approach to accessing classes.
* Raised the **level of the actor abstraction**.
* Targeting non-distributed systems experts. 
  * This level of indirection provides the runtime with the opportunity to solve many hard distributed systems problems that must otherwise be addressed by the developer, such as actor placement and load balancing, deactivation of unused actors, and actor recovery after server failures,
* Using local caches that map actor identity to its physical locations.
* **Virtual actors** is analogous to virtual memory, where a given logical memory page may be mapped to a variety of physical addresses over time, or even paged out. 
  
# Key Points

* High-scale interactive services demand high throughput with low latency and high availability, difficult goals to meet with the traditional stateless 3-tier architecture. The actor model makes it natural to build a stateful middle tier and achieve the required performance. However, the popular actor model platforms still pass many distributed systems problems to the developers.
* The Orleans programming model introduces the novel abstraction of virtual actors that solves a number of the complex distributed systems problems, such as reliability and distributed resource management, liberating the developers from dealing with those concerns. At the same time, the Orleans runtime enables applications to attain high performance, reliability and scalability.
* This paper presents the design principles behind Orleans and demonstrates how Orleans achieves a simple programming model that meets these goals. We describe how Orleans simplified the development of several scalable production applications on Windows Azure, and report on the performance of those production systems.
* Actors are units of isolation and distribution. Actors are isolated; they don't share memory. Two actors can communicate only by sending messages. 

## Single Threading

* Orleans ensures that at most one thread runs at a time within each activation. Thus, activation state is never accessed by multiple threads simultaneously, so race conditions are impossible and locks and other synchronization primitives are unnecessary
* By providing single-threaded access to the internal state of an actor instance. By not sharing data between actor instances except via message-passing.

## Virtual actors

* First, an Orleans actor always exists, virtually. It cannot be explicitly created or destroyed. **Its existence transcends the lifetime of any of its in-memory instantiations, and thus transcends the lifetime of any particular server.**
* Orleans actors are automatically instantiated: if there is no in-memory instance of an actor, **a message sent to the actor causes a new instance to be created on an available server**. An unused actor instance is automatically reclaimed as part of runtime resource management. 
* Third, the location of the actor instance is transparent to the application code, which greatly simplifies programming.
* **Fourth, Orleans can automatically create multiple instances of the same stateless actor, seamlessly scaling out hot actors.**

* **Perpetual existence**: Perpetual existence: actors are purely logical entities that always exist, virtually. An actor cannot be explicitly created or destroyed and its virtual existence is unaffected by the failure of a server that executes it.

* **Automatic instantiations**: Orleans’ runtime automatically creates in-memory instances of an actor called **activations**. An actor can have **zero or more activations.**
  * An actor will not be instantiated if there are no requests pending for it. When a new request is sent to an actor that is currently not instantiated, the Orleans runtime automatically creates an activation by picking a server, instantiating on that server the .NET object that implements the actor, and invoking its ActivateAsync method for initialization
* If the server fails, the runtime will automatically re-instantiates it on a new server on its next invocation. 
* An unused actor's in-memory instance is automatically reclaimed as part of the runtime resource management. 

## Automatic scaling out

* **Single activation mode** (default), in which only one simultaneous activation of an actor is allowed
* **Stateless worker mode**, in which many independent activations of an actor are created automatically by Orleans on-demand (up to a limit) to increase throughput. 
* Remove the need to explicitly activate or deactivate an actor, as well as supervise its lifecycle, and re-create it on failures.

## Fault tolerance

* Application responsible for checkpointing stateful progress

## Consistency

* No failure: Strong consistency. Failure (partition): Eventual consistency

## Promises

* Orleans method calls return immediately with a promise for a future result, rather than blocking until the result is returned.
* Initially, a promise is **unresolved**; it represents the expectation of receiving a result at some future time. When the result is received, the promise becomes **fulfilled** and the result becomes the value of the promise. If an error occurs, either in the calculation of the result or in the communication, the promise becomes **broken**.
* The main way to use a promise is to schedule a closure (or continuation) to execute when the promise is resolved. Closures are usually implicitly scheduled by using the await C# keyword on a promise.

## Turns

* Actor activations are single threaded and do work in chunks, called turns. An activation executes one turn at a time. A turn can be a method invocation or a closure executed on resolution of a promise. While Orleans may execute turns of different activations in parallel, each activation always executes one turn at a time.
* For simplicity, an activation does not receive a new request until all promises created during the processing of the current request have been resolved and all of their associated closures executed.
* While the turn-based asynchronous model allows for interleaving of turns for multiple requests to the same activation, the reasoniong about that is challenging. 
* Thus, an activation does not receive a new request until all promises created during the processing of the current request have been resolved and all of their associated closures executed.

## Persistence

* An actor class can declare a property bag interface that represents the actor state that should be persisted. 
* Up to application logic to decide when to checkpoint an actor's persistent state. 

## How does Orleans target low latency?

* Cooperative multitasking (remove context switching overhead). 
* Asynchronous IO
  * All methods and properties of an actor interface are asynchronous, returning promises for the results. 
  * Orleans method calls return immediately with a promise for a future result, rather than blocking until the result is returned. 
* Spawn on a machine that already has the data. 

## Actors
* No shared memory. All through message passing.

## Reminder

* A reminder is a timer that fires whether or not the actor is active. Thus, it transcends the actor activation that created it, and continues to operate until explicitly deleted. 
* If a reminder fires when its actor is not activated, a new activation is automatically created to process thereminder message, just like any other message sent to that actor.
  * Reminders are a useful facility to execute infrequent periodic work despite failures and without the need to pin an actor’s activation in memory forever.

## Implementations

* A cluster of servers in a datacenter.
* A server has three components:
  * Messaging;
    * The messaging subsystem connects each pair of servers with a single TCP connection and uses a set of communication threads to multiplex messages between actors hosted on those servers over open connections.
  * Hosting
    * Where to place activations and manage their life cycles.
  * Execution.
    * The execution subsystem runs actors’ application code on a set of compute threads with the single-threaded and reentrancy (an activation might be given another request to process in between turns of a previous request) guarantees.
* Cooperative multitasking (allow a thread run to termination) rather than preemptive (inefficient with large number of threads)
  * Orleans is not currently intended for a multi-tenant environment. 

## Orleans directory
* The Orleans directory is implemented as a one-hop distributed hash table (DHT) [17]. Each server in the cluster holds a partition of the directory, and actors are assigned to the partitions using consistent hashing.
* When a new activation is created, a registration request is sent to the appropriate directory partition. Similarly, when an activation is deactivated, a request is sent to the partition to unregister the activation.

# Limitations

One such pattern is an application that intermixes frequent bulk operations on many entities with operations on individual entities. Isolation of actors makes such bulk operations more expensive than operations on shared memory data structures. The virtual actor model can degrade if the number of actors in the system is extremely large (billions) and there is no temporal locality. **Orleans does not yet support cross-actor transactions, so applications that require this feature outside of the database system are not suitable.**

* Actor's persistent state is left to the programmer. 

* Eventual single activator consistency. (say in the case of a network partition)