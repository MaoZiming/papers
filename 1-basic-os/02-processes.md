# Processes

## Hardware Privilege Level

* Setting managed by the CPU to restrict types of operations that can be executed
* **X86**: four **privilege levels**
    * **Ring 0:** highest privilege level, generally used for most trusted functions of OS kernel
    * **Ring 1, Ring 2:** seldom use. Ring 1 is used for paravirtualization. 
    * **Ring 3:** lowest privilege levels, typically used for user-mode
* Connections
    * Kernel mode: typically high hardware privilege level (e.g., Ring 0)
    * User mode: typically low hardware privilege level (e.g., Ring 3).
  
## Process: abstraction for CPU

* CPU abstraction of *a running program*
* **What constitutes a process?**
    * Memory: content of memory in its address space
        * **Address space**: memory that the process can address
    * Register: content of CPU registers (PC, stack pointer, etc.)
        * PC: which instruction will execute next
        * SP: manage stack for function parameters
    * Storage: information about I/O (open files, etc.)
* **API**
    * E.x. creation
        * Allocate memory for runtime stack and heap
        * Lazy loading: loading pieces of code and data
        * Start at entry point (i.e. `main()`), transfer control of CPU to the process, execute
    * UNIX-like environment API
        * `fork()`: creates a new process, creator is parent, newly created process is child, child process is nearly identical copy of parent
        * `exec`: allows a child to break free from similarity to its parent and execute an entirely new program
        * `wait()`: allows a parent to wait for its child to complete execution
* **Process state:** running, ready, blocked
* **Mechanism:** ***limited direct execution***
    * V.s. direct execution: run program directly on the CPU
        * Problem: restricted operation, switching between processes
    * CPU support: **user mode** and **a privileged (non-restricted) kernel mode**
        * Kernel: OS
        * Use **system call** to **trap** into kernel to request OS services
        * The trap instruction saves register state carefully, change hardware status to kernel model, and jump into the OS to a pre-specified destination: **trap table**
        * After finish a system call, **return-from-trap**
        * Once program running, OS must use hardware mechanisms to ensure user program does not run forever: **timer interrupt** (i.e. **non-cooperative**)
            * Timer device programmed to raise interrupt
            * When raised
                * The current running process is halted
                * A pre-configured interrupt handler in the OS run

## Thread

* Thread
    * I.e. like a separate process, except they share the same address space
    * Multi-threaded program has more than one point of execution (i.e. multiple PCs)
    * **Why thread?**
        * Parallelism: utilize multi-core
        * To avoid blocking program due to slow I/O: e.x. server-based applications
        * Why not multi-process
            * Sharing data is easier
            * Processes are more logically separate tasks (little sharing)
    * **What constitute of a thread?**
        * A program counter (PC) and private set of registers
            * Save states of each thread of a process to thread control blocks (TCBs)
            * Context switch must take place (but no need to switch page table, for example)
        * A stack per thread
            * Thread-local storage
    * **Central to concurrent code: 4 problems**
        * **Critical section**: piece of code that accessed a shared resource
        * **Race condition**: arise if multi-thread enter critical section roughly at the same time; both attempts to update the shared data structure, lead to undesirable outcome
        * **Indeterminate program**: consists of one or more race conditions
            * Outcome is not deterministic
        * Threads should use **mutual exclusion** primitives
            * Guarantee that only a single thread enter a critical section, avoid races, and produce deterministic program output
    * **POSIX threads library**
        * `pthread_create()` and `pthread_join()`: create a thread
        * `pthread_mutex_lock()` and `pthread_mutex_unlock()`: lock
        * `pthread_cond_wait()`, `pthread_cond_signal()`: condition variables
    * POSIX: Portable Operating System Interface (for uniX?)
      * “everything is a file”
    
## Address Space: abstraction for memory

* Address space
    * Virtual address space
        * Illusion that each program has the view that it has a large contiguous address space to put its code and data into
        * Goal
            * Transparency: just like own private physical memory, OS does multiplex
            * Efficiency: time and space, need hardware support (i.e. TLBs)
                * Space: not spend too much memory for structures for virtualization
            * Protection: isolate among processes
    * Contains
        * Code: where instructions live
        * Data: global and stack variables. 
        * Stack: contains local variables arguments to routines, return values, etc.
        * Heap: contains malloc’d data, dynamic data structure

# Process

* **Instruction Set Architecture** (ISA): 
  * ISA defines the supported instructions, data types, registers, the hardware support for managing main memory (such as the memory consistency, addressing modes, virtual memory), and the input/output model of implementations of the ISA.
     * E.g. x86, developed by Intel, CISC
     * ARM, developed by ARM Holdings, RISC
* Process
  * Processor (Von Neumann model of computing):
  * Fetch an instruction from memory
  * Decode the instruction (Figure out what instruction it is)
    * During decoding, the processor needs to: determine what instruction is represented by the opcode; prefixes, and other values attached to the instruction and pass the relevant information to the relevant circuitry (in most modern processor designs) break the instruction down into its corresponding micro-operations With modern processors.
  * Execute the instruction
* Contents of CPU registers (including the program counter and stack pointer).
  * program counter (PC), or instruction pointer (IP): which instruction will execute next
  * stack pointer & frame pointer: manage the stack for function parameters, local variables, and return addresses
  * Stack pointer: tracks the top of the stack in memory
    * Stack holds temporary results
    * Permit recursive execution
  * Frame pointer: points to a fixed location within the stack frame of the current function. The stack frame contains the function’s local variables, parameters, and return address.
    * Redundant (can just use stack pointer + offset? save one extra register. )
    * Useful when getting stack trace. 
* Information about I/O.
  * persistent storage devices: I/O information for a list of files the process currently has open
* Process API:
  * Calls programs can make related to processes: creation, destruction, etc.
  * Create
  * Destroy
  * Wait
  * Miscellaneous Control (Suspend and resume)
  * Status
* Process states:
  * Running, ready-to-run, blocked
* Process List
  * Information about all processes that are ready and some additional information to track which process is currently running.
  * Each entry of the list is found in what is sometimes called Process Control block (PCB).
* Process —> a running program
  * **Sits** on a disk (with instructions, static data) in some kind of executable format.
* Operating system —> takes these bytes and gets them running
  * Makes the system easy to use, never concern with whether CPU is available
* Virutualizing CPUs
  * Mechanism: time sharing and context switch

## Process Creation

* ![image](images/02-processes/process-creation.png)

* In early (or simple) operating systems, the loading process is done eagerly, i.e., all at once before running the program;
* Lazy loading: loading pieces of code or data only as they are needed during program execution (i.e. paging, swapping)
* Before running the process: allocate memory for the program’s run-time stack (C programs use the stack for local variables, function parameters, and return addresses), also for the program’s heap (for explicitly requested dynamically-allocated data; other initialization related to I/O ). In Unix systems, each process by default has three open file descriptors, for standard input, output, and error [stored in heap.]
* Start the program at the entry point: `main()` routine, OS transfers control of CPU to the process, begin execution
* Interrupt vector, indexed by interrupt number. 
  * Entry of the vector contains the address and properties of each interrupt handler. 
* RTU: Return to user. 

# Limited Directed Execution

* The trap tables (with locations of trap handlers) must be set up by the OS at **boot time**, and make sure that they cannot be readily modified by user programs.
  * The timer interrupt. This approach is a non-cooperative approach to CPU scheduling.
* Direct Execution Protocol (Without Limits)
  * ![image](images/02-processes/direct-execution-protocol.png)
* System call:
  * trap: jumps to the kernel and raises privileged level to kernel mode
    * push the program counter, flag, and a few registers onto a per-process kernel stack
    * return-from-trap: returns to user program, reduces privilege level back
  * pop values off the stack and resume execution
  * system-call number: assigned to each system call
    * **protection**: user code cannot specify an exact address to jump to, but request service via number
  * Switching Between Processes
    * Cooperative Approach: wait for system calls
      * the OS regains control of the CPU by waiting for a system call (e.g. **Yield**) or an illegal operation of some kind (divide by zero, or tries to access memory that it shouldn't have, generating a trap) to take place
      * If a process gets stuck in an infinite loop, you can only reboot the machine.
    * Non-Cooperative Approach: the OS takes control
      * Use: timer interrupt and interrupt handler
      * A timer device can be programmed to raise an interrupt every so many milliseconds; when the interrupt is raised, the currently running process is halted, and a pre-configured interrupt handler in the OS runs.
      * During boot time, the OS must inform the hardware of which code to run when the timer interrupt occurs; thus, at boot time, the OS does exactly that. Second, also during the boot sequence, the OS must start the timer, which is of course a privileged operation. Once the timer has begun, the OS can thus feel safe in that control will eventually be returned to it, and thus the OS is free to run user programs.
    * Saving and restoring context
      * Context switch: save a few register values for current process (onto its kernel stack), and restore a few for the next process
  * Limited Direct Execution Protocol
    * ![image](images/02-processes/limited-direct-execution-protocol.png)
  * Two types of register save / restore
    * When timer interrupt occurs, user registers of the running process are implicitly saved by the **hardware**, using the **kernel stack** of that process
      * Switch from A to B, the kernel registers are explicitly saved by the **software** (i.e. the OS), but this time into memory in the process structure of the process
  * system call at the same time of timer interrupt?
    * one simple way: disable interrupt during interrupt processing

## Process States

* **Running**: running on a processor
* **Ready**: ready to run but OS has chosen not to run it at the moment
* **Blocked**: the process has performed some kind of operation that makes it not ready to run until some other event takes place (i.e. I/O request)
* Being moved from ready to running means the process has been scheduled; being moved from running to ready means the process has been descheduled.
* Once a process has become blocked (e.g., by initiating an I/O operation), the OS will keep it as such until some event occurs (e.g., I/O completion); at that point, the process moves to the ready state again (and potentially immediately to running again, if the OS so decides).
* These need a OS scheduler.
  * ![image](images/02-processes/process-state-transitions.png)
* Register context: hold the contents of its registers; when the process is stopped, its registers will be saved to this memory location
* Zombie: when the process has exited but has not yet been cleaned up
* When finished, the parent will make one final call (e.g., `wait()`) to wait for the completion of the child, and to also indicate to the OS that it can clean up any relevant data structures that referred to the now-extinct process.

## What OS tracks about a process

* Register context will hold, for a stopped process, the contents of its registers. When a process is stopped, its registers will be saved to this memory location; by restoring these registers (i.e., placing their values back into the actual physical registers), the OS can resume running the process.

# Process API

* Process ID (PID)
* `fork()`
  * The value it returns to the caller of fork() is different. Specifically, while the parent receives the PID of the newly-created child, the child receives a return code of zero.
* `exec()` family of system calls allows a child to break free from its similarity to its parent and execute an entirely new program.
  * Loads the code (and static data) from that executable and overwrite its current code segment (and current static data) with it; the heap and stack and other parts of the memory space of the program are re-initialized
  * Does not create a new process; transforms the currently running program into a different running program
  * Never returns if success; as if the old program never runs.
* `wait()`
  * Parent can wait for the child to finish executing, thereby finishing before the child.
* The separation of `fork()` and `exec()` enables features like input/output redirection, pipes and other cool features.
  * The separation of **fork()** and **exec()** is essential in building a UNIX shell, because it lets the shell run code after the call to fork() but before the call to exec(); this code can alter the environment of the about-to-be-run program, and thus enables a variety of interesting features to be readily built.
  * The way the shell accomplishes input/output redirection is quite simple: when the child is created, before calling exec(), the shell (specifically, the code executed in the child process) closes standard output and opens the file newfile.txt. By doing so, any output from the soon-to-be-running program is sent to the file instead of the screen
* Shell:
  * It shows you a prompt and then waits for you to type something into it. You then type a command (i.e., the name of an executable program, plus any arguments) into it; in most cases, the shell then figures out where in the file system the executable resides, calls fork() to create a new child process to run the command, calls some variant of exec() to run the command, and then waits for the command to complete by calling wait(). When the child completes, the shell returns from wait() and prints out a prompt again, ready for your next command
* Signals
  * Process control is available in the form of signals, which can cause jobs to stop, continue, or even terminate.
  * a process should use the signal() system call to “catch” various signals; doing so ensures that when a particular signal is delivered to a process, it will suspend its normal execution and run a particular piece of code in response to the signal.

### IPC

* Shared memory vs. message passing;


* Consider an in-memory queue.
* Data written by process A is held in memory until process B reads it. 
  * If producer (A) tries to write when buffer full, it blocks (Put sleep until space)
  * If consumer (B) tries to read when buffer empty, it blocks (Put to sleep until data)
* **Pipes** are an abstraction of a single queue
  * Used for communication between multiple processes **on one machine**
* **Sockets** are an abstraction of two queues, one in each direction
  * Used for communication between multiple processes on different machines

### Context Switch

* Switching between processes is significantly slower than switching between threads. 
  * PCB is much bigger than TCB. 
  * Even faster, switch user threads (in user space. `yield`).
    * User-level thread libraries. 
    * `yield` calls the library. Just a function call!
    * Many (user-level threads) to one (kernel-level thread). 

### Hyperthreading
* Two threads on the same core. 
* `vCPU` is a hyperthread.
* Intuition: one instruction doesn't use all components of the core. 
* Take advantage of that and to put a second thread. 


# Non-Uniform Memory Access (NUMA)

## Overview
NUMA is a memory design used in multiprocessor systems where the memory access time depends on the memory location relative to a processor. In NUMA, the system is divided into multiple nodes, each containing a processor or processors and memory. The memory is physically distributed, meaning each processor has its own local memory, but all processors can access all memory in the system.

### Key Features:
- **Local Memory:** Each processor can access its local memory faster than non-local memory (memory associated with other processors).
- **Distributed Memory:** Memory is spread across different nodes, but all memory is accessible by all processors.

## Benefits of NUMA
- **Scalability:** NUMA allows for better scaling in multiprocessor systems, as each processor has direct access to its memory, reducing bottlenecks.
- **Performance:** By optimizing memory access patterns, NUMA can offer better performance for workloads that are designed to take advantage of memory locality.

## Problems Associated with NUMA

### 1. **Memory Access Latency**
   - **Problem:** The time it takes to access memory can vary significantly depending on whether the memory is local to the processor or on a remote node.
   - **Impact:** If a process frequently accesses remote memory, it may experience higher latencies, leading to degraded performance.

### 2. **Load Balancing**
   - **Problem:** Uneven distribution of workloads can result in some nodes being overloaded while others are underutilized.
   - **Impact:** This imbalance can cause inefficient use of resources and reduce overall system performance.

### 3. **Memory Bandwidth Contention**
   - **Problem:** When multiple processors try to access the same memory node simultaneously, it can lead to contention and reduced memory bandwidth.
   - **Impact:** Contention can slow down memory access and negatively affect the performance of memory-intensive applications.

### 4. **Complex Programming**
   - **Problem:** Developers must explicitly manage memory placement and data locality to optimize performance on NUMA architectures.
   - **Impact:** This added complexity increases the development effort and the potential for errors, especially in large-scale systems.

### 5. **Operating System and Application Support**
   - **Problem:** Not all operating systems and applications are NUMA-aware, meaning they may not be optimized to take advantage of NUMA's benefits.
   - **Impact:** Non-NUMA-aware software may suffer from poor performance due to inefficient memory access patterns.

# Non-NUMA Systems (Uniform Memory Access - UMA)

## Overview of UMA (Uniform Memory Access)
In a UMA system, all processors share the same physical memory, and the memory access time is uniform regardless of which processor accesses the memory or where the memory is located. This means that any processor can access any memory location with the same latency, leading to a simpler memory architecture.

### Characteristics of UMA:
- **Uniform Access Time:** All processors experience the same memory access time, regardless of the memory location.
- **Shared Memory:** A single memory space is shared by all processors, with no distinction between local and remote memory.
- **Simpler Programming Model:** Since memory access times are consistent, developers do not need to manage memory placement or worry about memory locality.

### Examples of UMA Systems:
- **SMP (Symmetric Multiprocessing) Systems:** These systems typically employ a UMA architecture where multiple processors share a single, centralized memory.
- **Single-Processor Systems:** These inherently use UMA, as there is only one processor accessing the memory.
- **Older Multiprocessor Systems:** Many older multiprocessor systems are based on UMA, where the memory is not distributed across nodes.

### Benefits of UMA:
- **Simpler Design:** UMA systems are easier to design and program because memory access times are uniform.
- **Ease of Development:** Developers do not need to optimize for memory locality, simplifying application development.

### Drawbacks of UMA:
- **Scalability Issues:** As the number of processors increases, the memory bus can become a bottleneck, leading to reduced performance.
- **Limited Performance for Large Systems:** In large-scale systems, UMA can struggle with performance issues due to contention on the shared memory bus.

# Symmetric Multiprocessing (SMP)

## Overview
Symmetric Multiprocessing (SMP) is a parallel processing architecture where two or more identical processors are connected to a single, shared main memory and operate under a single operating system instance. In SMP systems, all processors have equal access to memory, I/O devices, and other system resources, allowing them to perform tasks independently while sharing the workload.

### Key Features of SMP:
- **Multiple Identical Processors:** All processors in an SMP system are identical and can execute any task or process.
- **Shared Memory:** All processors share the same physical memory space, and any processor can access any part of the memory with uniform latency.
- **Single Operating System Instance:** A single operating system manages all processors, memory, and I/O resources, providing a unified view of the system.
- **Load Balancing:** The system can dynamically balance the workload across all processors, helping to optimize performance and resource utilization.

## Benefits of SMP:
- **Improved Performance:** By distributing tasks across multiple processors, SMP systems can execute more instructions in parallel, leading to better performance, especially for multi-threaded applications.
- **Simplified Programming:** Developers can write applications without needing to manage memory placement or processor affinity, as the operating system handles resource allocation and scheduling.
- **Scalability:** SMP systems can scale up to a certain extent by adding more processors, although this is typically limited by the memory and I/O bandwidth.

## Drawbacks of SMP:
- **Scalability Limitations:** As the number of processors increases, contention for shared memory and other resources can become a bottleneck, limiting the system's ability to scale effectively.
- **Memory Bandwidth Contention:** All processors share the same memory bus, which can lead to contention and reduced performance as more processors are added.
- **Complex Cache Coherency:** Maintaining cache coherency (i.e., ensuring all processors have a consistent view of memory) becomes increasingly complex as the number of processors increases, potentially impacting performance.

## Common Use Cases:
- **Servers:** SMP is commonly used in server environments where workloads can benefit from parallel processing, such as web servers, database servers, and application servers.
- **Workstations:** High-performance workstations, especially those used for tasks like video editing, 3D rendering, and scientific simulations, often use SMP to leverage multiple processors.
- **Mainframes:** Large-scale mainframe systems may use SMP to handle extensive and diverse workloads efficiently.
