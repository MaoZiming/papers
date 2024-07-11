# OS Concept 

- **Types of kernel**
    - **Monolithic kernel: speed!**
        - E.x. Unix, Linux, BSD, Multics
        - All OS services operate in kernel space
            - Provides scheduling, file management through sys calls
        - Pros
            - Performance: communication between components is fast and efficient!
        - Cons
            - Complexity: huge
            - Poor maintainability: hard to debug and maintain
            - Poor reliability: if any service fails, system failure

- **Microkernel**: **modularity and reliability**
    - E.x. L4
    - Minimalistic approach
        - Kernel only keep: **basic IPC, virtual memory, scheduling**
          - Exokernel: only exposes resources.
          - Microkernel: mostly consists of IPC (Interprocess communication)
            - Services (like FS, Processes, even user progrmas) communicates to the microkernel through IPC.
            - Still has some ideas of processes and memory. 
        - MacOS (BSD Unix + Mach): monolithic on top of microkernel. Hybrid kernel. 
        - Put other services run in user space
            - Libraries running on user space
            - E.x. device drivers, networking, file system, etc.
        - Communication between components is provided by IPC (i.e. applications R/W through IPC)
    - Pros
        - Scalability: kernel smaller, separation of services at separate layers
        - Extensibility: new services can be easily added
        - Easy maintenance and debugging
        - More secure and reliable: a single service faults not lead to failures
    - Cons
        - Bad performance: lots of system calls (i.e. IPC) and context switches
    - File read —> communicate with the file system service residing in the user space through IPC. This involves a series of message exchanges and possibly more context switches
    - ![alt text](images/61-os-abstraction/comparison.png)

- **Hybrid kernel**
    - E.x. Windows, Android
    - Combine best of both worlds
        - Speed and simple design of a monolithic kernel
        - Modularity and stability of a microkernel
        - User-mode services: Some components, such as  subsystems (e.g., user-mode drivers and environment subsystems like the Windows Subsystem for Linux), run in user space. This separation helps improve system robustness and security.
    - Pros: still similar to monolithic kernel
        - With disadvantages similar

- **Exokernel**
    - Follow end-to-end principals
        - Extremely minimal: extreme version of microkernel
        - Fewest hardware abstractions as possible
        - Just allocates **physical resources** to apps
    - Pros: minimal and simple
    - Cons
        - More work for application developer
        - Poor isolation: each application implements own LibOS
            - Libraries accessing directly the HW, don’t have isolations
        - Hardware compatibility: need to change LibOS depending on HW interfaces

- **Unikernel** = Exokernel + Containers
    - Goal: link application with just enough OS functionality to allow it to execute
        * Instead of OS on top of VMM, use simple library kernel that get's optimized into a application specific binary
        * Dead code elimination so code size is smaller and simpler
        * Strongly typed library OS leads to fewer bugs and increased security
        * Faster overall since system calls become procedure calls over the VMM abstraction
        * Result: faster OS with less code and 50ms boot times
    - Key idea: run one application per VM, one process per application
        - Everything compiled into a VM image
    - Pros
        - extremely lightweight (~MB), tailored to perform just one specific application
        - customizable
        - better isolation
    - Cons
        - Recompile, each libraries are untrusted

### 1.2 Hardware Privilege Level

- Setting managed by the CPU to restrict types of operations that can be executed
- X86: four privilege levels
    - **Ring 0:** highest privilege level, generally used for most trusted functions of OS kernel
    - **Ring 1, Ring 2:** seldom use
    - **Ring 3:** lowest privilege levels, typically used for user-mode
- Connections
    - Kernel mode: typically high hardware privilege level (e.g., Ring 0)
    - User mode: typically low hardware privilege level (e.g., Ring 3).
  
## Process: abstraction for CPU

- CPU abstraction of *a running program*
- **What constitutes a process?**
    - Memory: content of memory in its address space
        - **Address space**: memory that the process can address
    - Register: content of CPU registers (PC, stack pointer, etc.)
        - PC: which instruction will execute next
        - SP: manage stack for function parameters
    - Storage: information about I/O (open files, etc.)
- **API**
    - Create, destroy, wait, miscellaneous control, status
    - E.x. creation
        - Allocate memory for runtime stack and heap
        - Lazy loading: loading pieces of code and data
        - Start at entry point (i.e. `main()`), transfer control of CPU to the process, execute
    - UNIX-like environment API
        - `fork()`: creates a new process, creator is parent, newly created process is child, child process is nearly identical copy of parent
        - `exec`: allows a child to break free from similarity to its parent and execute an entirely new program
        - `wait()`: allows a parent to wait for its child to complete execution
- **Process state:** running, ready, blocked
- **Mechanism:** ***limited direct execution***
    - V.s. direct execution: run program directly on the CPU
        - Problem: restricted operation, switching between processes
    - CPU support: **user mode** and **a privileged (non-restricted) kernel mode**
        - Kernel: OS
        - Use **system call** to **trap** into kernel to request OS services
        - The trap instruction save register state carefully, change hardware status to kernel model, and jump into the OS to a pre-specified destination: **trap table**
        - After finish a system call, **return-from-trap**
        - Once program running, OS must use hardware mechanisms to ensure user program does not run forever: **timer interrupt** (i.e. **non-cooperative**)
            - Timer device programmed to raise interrupt
            - When raised
                - The current running process is halted
                - A pre-configured interrupt handler in the OS run

## Thread: new abstraction for single running process

- Thread
    - I.e. like a separate process, except they share the same address space
    - Multi-threaded program has more than one point of execution (i.e. multiple PCs)
    - **Why thread?**
        - Parallelism: utilize multi-core
        - To avoid blocking program due to slow I/O: e.x. server-based applications
        - Why not multi-process
            - Sharing data is easier
            - Processes are more logically separate tasks (little sharing)
    - **What constitute of a thread?**
        - A program counter (PC) and private set of registers
            - Save states of each thread of a process to thread control blocks (TCBs)
            - Context switch must take place (but no need to switch page table, for example)
        - A stack per thread
            - Thread-local storage
    - **Central to concurrent code: 4 problems**
        - **Critical section**: piece of code that accessed a shared resource
        - **Race condition**: arise if multi-thread enter critical section roughly at the same time; both attempts to update the shared data structure, lead to undesirable outcome
        - **Indeterminate program**: consists of one or more race conditions
            - Outcome is not deterministic
        - Threads should use **mutual exclusion** primitives
            - Guarantee that only a single thread enter a critical section, avoid races, and produce deterministic program output
    - **POSIX threads library**
        - `pthread_create()` and `pthread_join()`: create a thread
        - `pthread_mutex_lock()` and `pthread_mutex_unlock()`: lock
        - `pthread_cond_wait()`, `pthread_cond_signal()`: condition variables
    

## Address Space: abstraction for memory

- Address space
    - Virtual address space
        - Illusion that each program has the view that it has a large contiguous address space to put its code and data into
        - Goal
            - Transparency: just like own private physical memory, OS does multiplex
            - Efficiency: time and space, need hardware support (i.e. TLBs)
                - Space: not spend too much memory for structures for virtualization
            - Protection: isolate among processes
    - Contains
        - Code: where instructions live
        - Stack: contains local variables arguments to routines, return values, etc.
        - Heap: contains malloc’d data, dynamic data structure
