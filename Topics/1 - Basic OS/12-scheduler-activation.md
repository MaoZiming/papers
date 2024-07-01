# Scheduler Activations: Effective Kernel Support for the User-Level Management of Parallelism (1991) 

Link: https://flint.cs.yale.edu/cs422/doc/sched-act.pdf

Read: June 16th, 12:55AM. 

The paper presents a mechanism called "scheduler activations" to improve the management of thread-level parallelism. It tries to address the limitations found in both user-level threads and kernel-level threads by offering a method for user-level thread libraries to communicate with the kernel scheduler. The paper outlines various techniques such as **upcalls** to communicate state information between the user and kernel levels and also discusses the notion of "virtual processors" provided by the kernel to the user-level threads for scheduling.


* The performance of kernel threads, although typically an order of magnitude better than that of traditional processes, has been typically an order of magnitude *worse* than the best-case performance of user-level threads. 
  * Kernel-level threads require a context switch, which involves changing a large set of processor registers that define the current memory map and permissions. It also evicts some or all of the processor cache.
  * With kernel threads, the program must cross an extra protection boundary on every thread operation, even when the processor is being switched between threads in the same address space. This involves not only an extra kernel trap, but the kernel must also copy and check parameters in order to protect itself against buggy or malicious programs. By contrast, invoking user-level thread operations can be quite inexpensive, particularly when compiler techniques are used to expand code inline and perform sophisticated register allocation. Further, safety is not compromised: address space boundaries isolate misuse of a user-level thread system to the program in which it occurs.
  * However, the difference isn't big if your threads are predominantly doing I/O operations, as those have to go through the kernel in any case.
  * User-level threads just require a small amount of bookkeeping within one kernel thread or process.
  * With kernel thread management, a single underlying implementation is used by all applications. To be general- purpose, a kernel thread system must provide any feature needed by any reasonable application; this imposes overhead on those applications that do not use a particular feature.
  * In contrast, the facilities provided by a user-level thread system can be closely matched to the specific needs of the applications that use it, since different applications can be linked with different user-level thread libraries.

* Each application knows exactly how many (and which) processors have been allocated to it and has complete control over which of its threads are running on those processors. 
* The operating system kernel has complete control over the allocation of processors among address spaces including the ability to change the number of processors assigned to an application during its execution.

* The parallel programmer, then, has been faced with a difficult dilemma: employ user-level threads, which have good performance and correct behavior provided the application is uniprogrammed and does no I/O, or employ kernel threads, which have worse performance but are not as restricted.

* Scheduler activation: control from the kernel to the address space thread scheduler on a kernel event -> use the activation to modify user-level thread data structures, to execute user-level threads. 

— Kernel threads block, resume, and are preempted without notification to the user level.
— Kernel threads are scheduled obliviously with respect to the user-level thread state.

- kernel threads are used by the programmer to express parallelism poor performance and poor flexibility and when user-level threads are built on top of kernel threads (poor behavior in the presence of multiprogramming and I/O). 

— The kernel allocates processors to address spaces; the kernel has complete control over how many processors to give each address space's virtual multiprocessor.
— Each address space’s user-level thread system has complete control over which threads to run on its allocated processors, as it would if the application were running on the bare physical machine.
— The kernel notifies the user-level thread system whenever the kernel changes the number of processors assigned to it; the kernel also notifies the thread system whenever a user-level thread blocks or wakes up in the kernel (e.g., on I/O or on a page fault). The kernel's role is to vector events to the appropriate thread scheduler, rather than to interpret these events on its own.

— The user-level thread system notifies the kernel when the application needs more or fewer processors. The kernel uses this information to allocate processors among address spaces. However, the user level notifies the kernel only on those subset of user-level thread operations that might affect processor allocation decisions. As a result, performance is not compromised; the majority of thread operations do not suffer the overhead of communication with the kernel.

- Each user-level thread is allocated its own user-level stack when it starts running [2]; when a user-level thread calls into the kernel, it uses its activation's kernel stack. The user-level thread scheduler runs on the activation's user-level stack.

- As the first thread executes, it may create more user threads and request additional processors. In this case, the kernel will create an additional scheduler activation for each processor and use it to upcall into the user level to tell it that the new processor is available. 

- Upcalls are the interface used by the scheduler activations system in the kernel to inform an application of a scheduling-related event.

- The crucial distinction between scheduler activations and kernel threads is that once an activation's user-level thread is stopped by the kernel, the thread is never directly resumed by the kernel. Instead, a new scheduler activation is created to notify the user-level thread system that the thread has been stopped. The user-level thread system then removes the state of the thread from the old activation, tells the kernel that the old activation can be reused, and finally decides which thread to run on the processor. By contrast, in a traditional system, when the kernel stops a kernel thread, even one running a user-level thread in its context, the kernel never notifies the user level of the event. Later, the kernel directly resumes the kernel thread (and by implication, its user-level thread), again without notification. By using scheduler activations, the kernel is able to maintain the invariant that there are always exactly as many running scheduler activations (vessels for running user-level threads) as there are processors assigned to the address space.

- Instead, we adopt a solution based on recovery. When an upcall informs the user-level thread system that a thread has been preempted or unblocked, the thread system checks if the thread was executing in a critical section. (Of course, this check must be made before acquiring any locks). if so, the thread is continued temporarily via a user-level context switch. When the continued thread exits the critical section, it relinquishes control back to the original upcall, again via a user-level context switch. At this point, it is safe to place the user-level thread back on the ready list. We use the same mechanism to continue an activation if it was preempted in the middle of processing a kernel event.
  
## V.s. user thread
Avoid unnecessary blocking and support processor allocation: 
* User-level thread system notifies kernel when it needs more or fewer processors (doesn’t happen very often)
* Kernel notifies the user-level thread system whenever kernel changes # of processors assigned or an I/O event occurs

## V.s. kernel thread 
Flexibility and performance: user-level thread system has control over which threads to run on the allocated v-processors, without trapping into the kernel. 

## V.s. Exokernel / Microkernel 
Exo-kernel: Like scheduler activations, exokernels aim to give more control to the application level for better resource management. Both try to minimize the kernel's role in making policy decisions, leaving that to the user level.

Microkernel: Scheduler activations share the microkernel's philosophy of minimizing kernel responsibilities and offloading more control to the user level. Microkernels, however, aim for complete modularity and separation of concerns, while scheduler activations work within the framework of a monolithic kernel to provide specific functionality.

## Limitations 
More complex to implement, upcall performance is limited, also there is not much performance analysis provided in the paper. 

User-level threads

++ imporve preformance and flexibility
run at user-level, threads managed by library linked to application
high pref; thread routines, context switch on order of a procedure call
customizable to application needs (specialization)
-- OS unaware of user-level threads
kernel events hidden from user-level
When I/O, page faults block the user thread, the kernel thread serving as its virtual processor also blocks. As a result, the processor is idle, even if there are other user threads that can run.
kernel threads scheduled without respect to user-level state
Multiprogramming: when OS takes away kernel threads, it preempts kernel threads without knowledge of which user threads are running
Performance: may interact poorly with user-level scheduler
correctness: poor OS choices could even lead to deadlock
Kernel-level threads

++ OS aware of kernel threads, integrated w/ events & scheduling
-- but, high cost
high overhead: need syscall to invoke thread routine, context switch
too general: have to support all applications, adds code and complexity
scheduling used as an example (preemptive priority vs FIFO)

Kernel Level Threads Pros/Cons
Good functionality, system wide integration
Threads are seen and scheduled only by the kernel. A lot of kernel information should be invisible to user thread and can be useful for scheduling
Poor performance, every thread_related call traps. This situation is a lot worse in the 1990s than it is now mainly due to clock speed. The scheduling quanta are roughly the same, but because the clock speeds are much faster today, you can execute orders of magnitude more instructions per quanta today than you could in 1990. Even if traps, let’s say, costs 10 cycles to complete, it would be a much bigger fraction of the quanta in 1990 than it is today.
User Level Threads Pros/Cons
Good performances. (most threads operations don’t involve kernel)
Good scheduling policy flexibility: done by thread lib
Poor system-wide integration
Multi-programmed workloads are hard to schedule
I/O, page faults invisible
Potential for incorrect behavior
User level scheduler may not be cooperative. With user threads running on kernel threads, it may be that kernel threads block when a user-thread blocks, thus an application can run out of kernel threads to run their user threads.May be gilding the lily.
Some Problems about User-Level Threads on Kernel Interface
Insufficient visibility between the kernel and user thread lib
Kernel event such as pr-emption or I/O are not visible to user lib
For example, if user level threads block, then the kernel thread serving it also blocks.
Kernel threads are scheduled with respect to user-level thread library, we can have this interferences between two schedulers.
Kernel time-slicing of threads
For example, user level threads holding a spin-lock can be pre-emptied, which can potentially cause all other user threads to wait.
Scheduler Activation
The basic principle about scheduler activation is to expose revocation: telling me when you take something away. This is basically the same idea as the exokernel. For example, interfaces like

add_processor()
has_blocked()
The basics about scheduler activation are

Multi-threaded programs are still given an address space
Facilitate flow of kernel information between user and kernel threads
Kernel explicitly vectors kernel events to the user-level thread
via scheduler activation (upcall)
Extended kernel interface for processor allocation-related events
Essentially exchanging information

## Baselines

- **Approach 1: processes + shared memory**
    - Simple, uni-threaded model
    - Security provided by address space boundaries
    - But high cost for context switch, limits degree of concurrency
- **Approach 2: kernel-level threads**
    - One-to-one
    - Threads are exported by the kernel
    - `pthread_create()` (user space) —> traps into kernel through system call
    - State of a thread maintained in the kernel
    - Cons: performance limitations, lack of portability and flexibility
        - Overheads due to general implementation and cost of kernel traps
            - Context switch time better than process switch time, but still worse than user-level thread
        - System scheduler unaware of user-thread state, leading to lower utilization
- **Approach 3: “lightweight” (user-level) threads**
    - Many-to-one
    - `pthread_create()` (user space) —> no kernel trap
    - Thread semantics defined by application
    - Pros: performance, flexibility
        - Fast context switch time (i.e. within an order of magnitude of procedure call)
    - Cons: functionality
        - Unnecessary blocking (I/O, page faults, etc.)
            - OS may block entire process if one user-level thread blocks an I/O

## Key Insights

- Applications
    - Has little knowledge or influence over critical kernel-level events
- Kernel
    - Knows little about user-level scheduling information (e.g. parallelism) to make optimal scheduling decisions

## Key Techniques: N to M

- **Solution:** a mechanism that facilitates exchange of information between user- and kernel-level mechanisms
    - A set of library routines for user-level thread managements
    - Exposure of relevant kernel events to user-level thread management
- Some features
    - Kernel has control over how many physical processors to give to a process’s virtual multi-processor
        - Map N number of application threads to M number of kernel entities, or “virtual processors”
    - User-level thread system has control over which threads to run on the allocated v-processors
    - Kernel notifies the user-level thread system whenever kernel changes # of processors assigned or an I/O event occurs
    - User-level thread system notifies kernel when it needs more or fewer processors (doesn’t happen very often)
    - Application programmers sees no difference except perf. when working directly with kernel threads
- Some issues: kernel might interrupt a user-thread holding a lock —> deadlock
    - Sol: kernel informs a scheduler that a thread has been preempted, it checks to see if it was in a critical section, if so let it run out then preempt (i.e. user-level context switch)
- How is it done?
    - The kernel provides an application with a set of virtual processors (LWPs)
    - The application can schedule user threads onto an available virtual processor
    - Four upcall points
        - Add a processor
        - Processor preempted
        - Schedule activation blocked
        - Schedule activation unblocked
    - From user to kernel
        - Add more processors
        - Processor idle
    - Upcall: the kernel must inform an application about certain events
        - handled by thread library with an upcall handler, and this upcall handler must run on a virtual processor
    - When an application thread is about the block
        - The kernel makes an upcall to the application informing it that a thread is about to block and identifying the specific thread
        - The kernel allocates a new virtual processor to the application
        - The application runs an upcall handler on this new virtual processor, that saves the state of the blocking thread and relinquishes the virtual processor on which the blocking thread is running
        - Another thread that is eligible to run on the new virtual processor is scheduled then by the upcall handler
    - Done with blocking
        - The kernel makes another upcall to the thread library informing it that the previously blocked thread is now eligible to run
        - A virtual processor is also required for the upcall handler for this event, and the kernel may allocate a new virtual processor or preempt one of the user threads and then run the upcall handler on its virtual processor
        - The application schedules an eligible thread to run on an available virtual processor, after marking the unblocked thread as eligible to run

## Limitations

- More complex to implement, because both changes to kernel and user-space code are required

## Nice system principals

- **A general system design problem:** communicate information and control across layer boundaries while preserving the inherent advantages of layering, abstraction, and virtualization