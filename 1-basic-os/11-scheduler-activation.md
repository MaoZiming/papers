# Scheduler Activations: Effective Kernel Support for the User-Level Management of Parallelism (1991) 

Link: https://flint.cs.yale.edu/cs422/doc/sched-act.pdf

Read: June 16th, 12:55AM. 

- **Scheduler Activation**
  - User-level threads to communicate with kernel scheduler
- **upcalls**
  - Used by scheduler activation. 
  - Communciate staet information between the user and kernel. 
- **virtual processors**

## Kernel vs. User threads

### Kernel Thread
- Kernel threads have been typically an order of magnitude *worse* than user-level threads. 
  - Kernel threads (`pthread_create`): context switch, which involves changing a large set of processor registers that define the current memory map and permissions
  - Need a syscall to invoke a kernel routine. 
  - Even when the processor is being switched between threads in the same address space, require kernel mode. 
    - Extra kernel trap; the kernel must copy and check parameters to protect against buggy or malicious programs.
    - With kernel thread management, a single underlying implementation is used by all applications. 
    - Each thread needs a whole TCB. 
    - Overheads due to general implementation and cost of kernel traps
        - Context switch time better than process switch time, but still worse than user-level thread
    - System scheduler unaware of user-thread state, leading to lower utilization
  
### User thread
  - Invoking user-level thread (Python `threading` module) operations can be quite inexpensive. Context switch is low overhead similar to a **procedure call**. 
    - Further, safety is not compromised: address space boundaries isolate misuse of a user-level thread system to the program. 
    - However, I/O operations have to go through the kernel in any case. User-level threads have poor performance.
    - User-level threads just require a small amount of bookkeeping within one kernel thread or process.
    - Different applications have different user-level thread libraries. More customizations. 
    - When I/O, page faults block the user thread, the kernel thread serving as its virtual processor also blocks. As a result, the processor is idle, even if there are other user threads that can run. 
    - Kernel threads scheduled without respect to user-level state
    - **Multiprogramming**: when OS takes away kernel threads, it preempts kernel threads without knowledge of which user threads are running
    - **Performance**: kernel events may interact poorly with user-level scheduler
    - **Correctness**: poor OS choices could even lead to deadlock
  
### Problem:

  - User-level threads have good performance and correct behavior provided the application is uniprogrammed and does no I/O, while employ kernel threads, which have worse performance but are not as restricted.
    - Kernel threads block, resume, and are preempted without notification to the user level.
    - Kernel threads are scheduled obliviously with respect to the user-level thread state.

### Proposed Solution: 

- Scheduler activation: 
  - Upon kernel events (change of the number of processors, I/O events, a new processor available), use the activation to notify and modify user-level thread data structures, to execute user-level threads. 
  - Each address space’s user-level thread system has complete control over which threads to run on its allocated processors
  - The kernel notifies the user-level thread system whenever the kernel changes the number of processors assigned to it; the kernel also notifies the thread system whenever a user-level thread blocks or wakes up in the kernel (e.g., on I/O or on a page fault). 
  - The user-level thread system notifies the kernel when the application needs more or fewer processors. The kernel uses this information to allocate processors among address spaces. 
    - However, the user level notifies the kernel only on those subset of user-level thread operations that might affect processor allocation decisions. 
  
> Upcalls are the interface used by the scheduler activations system in the kernel to inform an application of a scheduling-related event.

  - The crucial distinction between scheduler activations and kernel threads is that once an activation's user-level thread is stopped by the kernel, the thread is never directly resumed by the kernel. Instead, a new scheduler activation is created to notify the user-level thread system that the thread has been stopped. The user-level thread system then removes the state of the thread from the old activation, tells the kernel that the old activation can be reused, and finally decides which thread to run on the processor. 
  - By contrast, in a traditional system, when the kernel stops a kernel thread, even one running a user-level thread in its context, the kernel never notifies the user level of the event. Later, the kernel directly resumes the kernel thread (and by implication, its user-level thread), again without notification. 
  - By using scheduler activations, the kernel is able to maintain the invariant that there are always exactly as many running scheduler activations (vessels for running user-level threads) as there are processors assigned to the address space.
  - When an upcall informs the user-level thread system that a thread has been preempted or unblocked, the thread system checks if the thread was executing in a critical section. If the thread is continued temporarily via a user-level context switch. When the continued thread exits the critical section, it relinquishes control back to the original upcall, again via a user-level context switch. At this point, it is safe to place the user-level thread back on the ready list. 
  - Still a monolithic kernel, different from microkernel or exokernel.
  - When an application thread is about the block
      - The kernel makes an upcall to the application informing it that a thread is about to block and identifying the specific thread
      - The kernel allocates a new virtual processor to the application
      - The application runs an upcall handler on this new virtual processor, that saves the state of the blocking thread and relinquishes the virtual processor on which the blocking thread is running
      - Another thread that is eligible to run on the new virtual processor is scheduled then by the upcall handler

### Limitations 
- More complex to implement, upcall performance is limited, also there is not much performance analysis provided in the paper. 
