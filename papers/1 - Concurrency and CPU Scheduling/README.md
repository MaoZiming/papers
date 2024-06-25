# Multiprogramming 
Reference very useful summary 
* https://github.com/parasj/papers/tree/master/multiprogramming
* https://pages.cs.wisc.edu/~sschang/OS-Qual/process/process_and_synchronization.htm
* https://w3.cs.jmu.edu/kirkpams/OpenCSF/Books/csf/html/IPCModels.html

Running processes on multicore, multiprocessor systems require coordination, conflict resolution, etc. How to design such systems? How should OS manage memory and schedule tasks? 

## Fundamental Abstraction 
* Process = set of threads + collection of resources
* Thread = a single execution sequence within a process

## Programming model: event v.s thread 
I find this article very interesting: https://berb.github.io/diploma-thesis/original/043_threadsevents.html. 

In terms of processes, and synchronization and communication, there is extensive debate about message-oriented systems v.s procedure-oriented system (i.e. like our discussions above). Message-oriented systems resemble event-driven systems, while procedure-oriented systems correspond to thread-based systems. They are duals of each other! 

What does this imply? the comparisons are not the models themselves, rather it is the implementation, applicability, and suitability based on the actual application requirements. 


### Thread Model 
In a thread-based system, each task or job runs in its own thread. Threads are like mini-programs that can run concurrently, making it seem as if multiple operations are happening at the same time. Typically deloys preemptive scheduling in thread based system, where OS decides when to switch tasks. 

**Pros:** easy to program 
* natural extension of sequential programming
* take advantage of real CPU concurrency and multi-cores 
* simple and powerful when tasks are mostly isolated and only share a limited amount of state (cf. multi-threaded web servers), typically for compute-bound tasks

**Cons:** 
* difficult to develop, understand and debug
* performance
  * coordination and synchronization is hard when shared states
  * overhead gets worse with large # of threads
    * cache and TLB misses, scheduling overhead    
    * large amount of memory required due to thread stack
    * lock: contention, deadlocks, live locks   

### Event Model 
In an event-driven system, there is a single thread with an event loop. The event loop continually checks for events (like incoming data) and calls the appropriate function (often called a "callback") to handle each event.

Instead of blocking, the program registers a callback function that the event loop should call when the I/O operation is complete. While waiting for this to happen, the event loop can handle other tasks or events. When the I/O operation is finally complete, the event loop simply calls the previously registered callback function to process the data. This yields the illusion of multi-threaded concurrency, because multiple conceptual flows of execution appear to happen at the same time. 

Cooperative scheduling: in an event-driven system, a task (or "event handler") will run to completion before the next task starts. It's "cooperating" by not hogging the system's resources for too long.

**Pros:**
* Fine-grained control: allow more explicit control over when tasks are scheduled and executed
* Typically for I/O bound task, like handling thousands of simultaneous connections in a web server

**Cons:**
* Complexity in code structure: a series of callback functions make the control flow harder to understand
* State ripping: developer responsibility to handle state between handlers (v.s. thread stack)

### Papers 
SEDA proposes a staged event-driven hybrid architecture for highly concurrent internet services. Applications are divided into "stages," each of which has an associated event queue. Threads within each stage process events from this queue, effectively blending events and threads. SEDA a dynamic resource controllers to allocate threads at each stage. 

Cappricios focus on a highly optimized threading libraries that employ compiler optimizations for thread stack sizes and adaptive scheduling. It borrows some aspects from event-driven models (like user-level threads and cooperative scheduling) to achieve these goals.

## Thread Abstraction: user v.s kernel 
There are two types of threads, user and kernel-level. Their trade-offs are demonstrated as follows. 

### User threads (many-to-one) 
User threads are managed by user level libraries (e.x. scheduling, sync), can be easily implemented by the users.

**Pros**: 1) Implementation is easier, simple, and quick to create 2) It is also fast and efficient: don't need to make system call, context switch is less. 

**Cons**: 1) Functionality: OS blocks entire process if one user-level thread blocks an I/O 2) OS has no insights on application needs, might make poor scheduling decisions 

### Kernel threads (one-to-one) 
**Pros**: OS sees and understand threads, sync, blocking, etc. 

**Cons**: 1) Performance limitations: must go to OS for all operations (i.e. creation, synchronization, scheduling) 2) Lack of portability and flexibility: may have limits on # of threads, etc.

### Papers
Scheduler Activations has a hybrid approach to enable exchange of information between user and kernel, and the main idea is to coordinate user level and kernel level scheduling through scheduler activations. 
