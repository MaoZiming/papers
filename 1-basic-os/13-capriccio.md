# Capriccio: Scalable Threads for Internet Services (2003) 

Read: June 16th, 10pm, 2024

Link: http://capriccio.cs.berkeley.edu/pubs/capriccio-sosp-2003.pdf

This paper proposes Capriccio, a scalable thread package for use with high-concurrency servers. The main proposal is to use **user-level threads** instead of event-based models (i.e. SEDA) to achieve high performance without sacrificing the ease of programming. 

Its approach is three-fold: 

* **Scalability**: Capriccio uses user-level threads coupled with cooperative scheduling to manage many threads efficiently. User-level threads are preferred over kernel threads for their flexibility and performance advantages,

* **Linked Stack**: Traditional stack allocation methods are not suitable for programs managing many threads. **Capriccio uses a dynamic stack allocation technique** that employs **linked chunks**, **which are managed through compile-time analysis and runtime checks.**

* **Resource-aware Scheduler**: The scheduler views the application as a sequence of stages, separated by blocking points, forming a "**blocking graph**". This graph is used to make application-specific scheduling decisions to maximize resource utilization without reaching bottlenecks.

## Limitations 
* Does not completely eliminate kernel crossings.
* Performance not necessarily better than event-based systems.

## Key Insight

### Problem

Same motivation as SEDA: Internet services have ever-increasing scalability demands 

* Thousands of simultaneous connections
* We need a programming model to achieve efficient and robust servers with ease

### Baseline: Event-based Programming Model

* Handle requests through a pipeline of stages, each request represented by an event, and each stage is implemented as event handler
* E.x. SEDA
* Pros
    * More efficient
        * V.s. context switching and locking overheads with threads
    * Threads are hard to program (deadlocks, synchronization)
    * Poor thread support (portability, debugging)
* Cons
    * Many event systems invoke a method in another module by sending a "call" event and waiting for a "return" event in response
    * Readability is a huge thing!
        * Programmer has difficulty to understand cause-effect ("call", "return pairs) when examining source code and when debugging, and these "call" and "return" are in different parts of the code. 
    * Stack ripping
        * Creating these call/return pairs require the programmer to manually save and restore live state

### Stack Ripping

In traditional synchronous programming:
```cpp
void processRequest() {
    int data = readFromNetwork();
    processData(data);
    writeToDatabase(data);
}
```
In event-driven architecture with stack ripping:
```cpp
void processRequest() {
    initiateReadFromNetwork();  // Non-blocking call
}

void onNetworkReadComplete(int data) {
    processData(data);
    initiateWriteToDatabase(data);  // Non-blocking call
}

void onDatabaseWriteComplete() {
    // Finalization steps
}
```
In this "ripped" version:
* The initial function `processRequest()` initiates a non-blocking network read.
* When the network read is complete, the system triggers `onNetworkReadComplete()`, which processes the data and initiates a non-blocking write to the database.
* Finally, once the database write is complete, the system calls `onDatabaseWriteComplete()` to finish the operation.

* Key aspects of stack ripping
  * Manually manage the state of the operations across multiple callbacks or event handlers
  * More complex and harder-to-maintain code.
* Syntactic sugar: `async/await` : performs stack ripping behind the scene. It abstracts and automates the process of stack ripping, making it more manageable and less error-prone for developers.

### Main Proposal

* “Any apparent advantages of events are better viewed as arguments for app-specific optimization and need for efficient thread runtime”

Just improve the thread runtime! 

Goals:

* Allow high performance without high complexity 
* Support existing thread’s API (POSIX)
* Scalability to **100,000** threads
* Flexibility to app-specific needs 
* Little or no modification to the application itself 

## Key Approach

There are three parts:

* Scalability
    * **User-level threads** + **cooperative** scheduling 
    * This is the main technique for addressing the overhead of **context switching**. 
    * Since Capriccio manages threads within user space, **the context switch operations do not require expensive system calls or kernel interventions.**
* Linked stack 
    * Stack allocation for large # of threads
    * **Compile-time analysis + runtime checks** 
* Resource-aware scheduler 
    * Application-specific scheduling based on predicted resource 

### #1: User-level Threads

* Why user-level threads?
    * Kernel threads: for enabling true concurrency via multiple devices, disk requests, CPUs
    * User threads: logical threads that provide a clean programming model
        * Pros
            * ***Flexibility***
                * Decouple from OS / kernel
                    * Allow faster innovations on both sides
                    * E.x. kernel async I/O mechanisms can be used without changing the applications
                * Address application-specific needs
                    * Increase flexibility of thread scheduler
                * Kernel threads cannot tailor the scheduling algorithm to fit a specific application. Instead, the user-level thread scheduler can be built along with the application.
            * ***Performance***
                * Greatly reduce overhead of thread synchronization
                    * E.x. single CPU, neither user threads nor thread scheduler can be interrupted during a critical section
                * **Do not require kernel crossings for mutex acquisition or release**
                    * V.s. kernel thread: kernel crossing for every sync operation
                * Memory management is more efficient
                    * V.s. kernel thread
                        * require data structure (TCB) that eat up kernel addr space
                        * decrease available space for I/O buffers, file descriptors, and other resources
        * Cons
            * Some times more kernel crossings
                * To retain control of processor when user-level thread executes a blocking I/O call, user-level thread package override blocking call with non-blocking equivalents
                * E.x. non-blocking network I/O `epoll` involves
                    * First polling sockets for I/O readiness
                    * Then performing actual I/O call
                * The actual I/O call is identical to the non-blocking version.
            * Must introduce a wrapper layer that translate blocking I/O to non-blocking ones, which leads to overhead
                * E.x. overhead significant for quick operations like in-cache reads
        * Benchmark: Benefits outweigh drawbacks
* Main thing
    * Cooperative scheduling
    * **Async I/O**
    * Efficient thread operations - O(1)

### Why does user-level threads do not require the kernel to manage or schedule them.
* No kernel involvement. User-level threads do not require the kernel to manage or schedule them. Context switching between these threads happens entirely in user space without needing to invoke the kernel's scheduler. This avoids the overhead of switching between user mode and kernel mode, which is required for switching kernel-level threads.

### #2: Linked Stack

### Use weighted call graphs (whole program analysis); each function is a node; each edge is a function call; node weights are calculated with stack frames. 

* Problem: conservative stack allocation per thread are unsuitable for programs with many threads
    * Abstraction of unbounded call stack for each thread
    * While in reality stack bounds are chosen conservatively large
    * E.x. 1GB virtual memory with just 500 threads
      * For example, LinuxThreads allocates two megabytes per stack by default; 
* Most threads consume only a **few kilobytes of stack space** at any given time
  * Significantly reduce the size of virtual memory dedicated to stacks if we adopt a **dynamic stack allocation policy** wherein stack space is allocated to threads on demand in **relatively small increments** and is deallocated when the thread requires less stack space.
    * Stack grows and shrinks dynamically based on the needs of the thread. 
* Idea: dynamic stack allocation with **linked chunks**
    * Alleviates VM pressure and improve paging behavior
* Method: compile-time analysis and checkpoint injection
    * **Small non-contiguous stack chunks** grow and shrink at runtime
    * Compiler analysis and runtime checks
        * Goal: place a reasonable bound on stack space consumed by each thread
        * Generate a ***weighted directed call graph***
    * Features
        * Each node is a call site annotated with max stack space for that call
        * Edges: function calls in-between nodes
        * Path: sequence of stack frame (length = sum of weights of all nodes)
        * Checkpoints are inserted at edges (call sites)
            * A checkpoint is a small piece of code that determines whether there is enough stack space left to reach the next checkpoint without causing stack overflow. 
            * If not enough space remains, a new stack chunk is allocated, and the stack pointer is adjusted to point to this new chunk. 
            * When the function call returns, the stack chunk is unlinked and returned to a free list.
            * Checkpoint placement
                * Break cycles (i.e. recursive frame)
                * Scan nodes to ensure path between checkpoints within desired bound (set as compile-time parameter)

### #3: Resource-aware Scheduler


### Blocking graphs: node is a location in program that is blocked. Node is composed of call chain used to reach blocking point. Resource-aware scheduling: edges are annotated with average running time - time it took the thread to pass between nodes. Nodes are annotated with resources used on its outgoing edge. 

> In this sense, Capriccio’s scheduler is quite similar to an event-based system’s scheduler. Our methods are more powerful, however, in that they deduce the stages automat- ically and have direct knowledge of the resources used by each stage, thus enabling finer-grained dynamic scheduling decisions.

### Basically fine-grained graphs (down to function calls `main`, `thread_create`) allow more fine-grained resource-aware scheduling. It also adapts to the current resource usage of the system. 

* Propose: application-specific scheduling for thread-based applications
* Key abstraction: ***blocking graph***
    * ![alt text](images/13-capriccio/blocking-graph.png)
    * Each node is a *location* in the program that is *blocked*. 
    * View applications as sequence of stages separated by blocking points
    * Generated at runtime
    * Nodes: program locations where threads block
    * **Edges: consecutive blocking points**
        * Annotated by
            * Average time taken to traverse an edge
            * Change in resource usage like memory, stack space, sockets
* General idea: schedule threads s.t. for each resource, utilization is increased until max throughput and then throttled back
    * Determine which resources are near their limits.
    * Predict the impact on resources if threads at a *particular node* were scheduled (since edges are annotated)
    * Prioritize nodes (and thus threads) based on resource needs and availability.
* Implement using separate run queues for each node
    * Periodically determine relative priorities of each node
    * Select a node by stride scheduling (i.e. decide which group of threads should be scheduled next)
    * Select threads within nodes by dequeueing from node’s run queues and scheduled for execution
* Pitfalls
    * Maximum capacity of particular resource is hard to determine
    * Thrashing not easily detectable
    * Non-yielding threads lead to unfairness and starvation in cooperative scheduling
    * Blocking graphs are expensive to maintain (i.e. 8% of execution time)

## Limitations

* Some future works not addressed by the paper 
    * Reduce kernel crossings with batching async network I/O 
    * Disambiguate function pointers in stack allocation
    * Improve resource-aware scheduling
        * Better detection of thrashing 
        * Tracking variance in resource usage 
* Performance not comparable to event-based systems 
* High overhead in stack tracing
* Lack of sufficient multi-processor support