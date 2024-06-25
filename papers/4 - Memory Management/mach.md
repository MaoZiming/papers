# The Duality of Memory and Communication in the Implementation of a Multiprocessor Operating System (1987) 
Reference:
* http://infolab.stanford.edu/~manku/quals/summaries/spiller-duality.htm
* https://github.com/parasj/papers/blob/master/multiprogramming/mach_os_duality_of_virt_mem_and_communication.md

Mach is an early predecessor of microkernel, providing the functionality of UNIX when desired but is generally lighter weight. It is a object-oriented semi-user-extensible memory system. 

In Mach, memory is viewed as **"abstract objects"** comprising collections of bytes with actions. It is easier for developers to extend system functionalities. For instance, file systems are managed through these "memory objects," and operations like reading and writing files are performed via these objects. **File systems are managed through memory objects.** 

## Semantics 
_Memory Objects_: memory abstracted as objects with actions 

_Tasks and Threads_: tasks are similar to processes in UNIX, and each task can have multiple threads. Tasks allocate memory by mapping to these memory objects, which can be shared. 

_Ports and Messages_: each memory object has an associated port. Tasks and threads communicate with these objects through ports using messages (i.e. data, pointers, or commands for RPCs).
* E.x. File reading
    * application sends IPC message to FS service in user space
    * FS service read data from disk (i.e. communicate with disk driver through IPC)
    * Monolithic: syscall  

_Kernel's Role_: protection, page lookup, Copy-on-Write (CoW) support, and IPC (i.e. system call like `pipe()`, `msgget()`, `msgsnd()`, `msgrcv()`). It also manages paging and cache, interfacing with memory objects when page faults occur or when data needs to be written back to secondary storage.

## V.s. Microkernel 
Mach focus on putting minimal functionalities into the kernel itself. 
* Kernels: handle IPC, low-level memory management, and protection
* User space: higher-level services, like file systems, device drivers, and even some aspects of memory management, are implemented in user space 

