# OS Concept 
- Kernels, process, threads, address spaces, and hardware privilege level (also known as dual-mode operation)
- APIs in a UNIX-like environment for using processes (fork, exec, and wait), threads (pthreads), and files (open, read, etc.)


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
        - Start at entry point (i.e. main()), transfer control of CPU to the process, execute
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

### 1.1 Kernel
An overview to different types of kernel is [here](https://github.com/lynnliu030/os-prelim/tree/main/system_structure). 

### 1.2 Hardware Privilege Level

- Setting managed by the CPU to restrict types of operations that can be executed
- X86: four privilege levels
    - **Ring 0:** highest privilege level, generally used for most trusted functions of OS kernel
    - **Ring 1, Ring 2:** seldom use
    - **Ring 3:** lowest privilege levels, typically used for user-mode
- Connections
    - Kernel mode: typically high hardware privilege level (e.g., Ring 0)
    - User mode: typically low hardware privilege level (e.g., Ring 3).

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

## File API

- `open`
- `read`
- `write`
- `close`
