# Basic Concepts Overview
### I. [Operating System Concepts and Abstractions](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/os_abstraction.md)

- Kernels, process, threads, address spaces, and hardware privilege level (also known as dual-mode operation)
- APIs in a UNIX-like environment for using processes (fork, exec, and wait), threads (pthreads), and files (open, read, etc.)


### II. [Synchronization](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/sync.md)

- Implementation of threads and locks (disabling interrupts and atomic operations like CompareAndSwap)
- How to use semaphores, locks, and condition variables

### III. [CPU Scheduling](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/cpu_sched.md)

- Classic policies: FCFS (First-Come, First-Served), Round Robin, Priorities, SRTF (Shortest Remaining Time First), MLFQ (Multi-Level Feedback Queue), and EDF (Earliest Deadline First)
- Starvation (including priority donation/inheritance) and deadlock (including prevention techniques)
- Look at how multi-core scheduling works as well

### IV. [Memory Management](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/memory_mgmt.md)
- Software support for address translation: page fault handler and multi-level page tables
- Hardware support for address translation: cache, TLB, and MMU
- Demand paging eviction policies: MIN (Belady's Algorithm), LRU (Least Recently Used), NRU (Not Recently Used, also known as the "Clock Algorithm"), and VAX/VMS second-chance lists
- Spatial locality, temporal locality, working sets, Belady's anomaly, and thrashing


### V. [Input / Output (I/O) and System Performance](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/io.md)
- Conceptual understanding of buses, controllers, interrupts, polling, programmed I/O (PIO), and Direct Memory Access (DMA)
- Addition: I/O schedulers


### VI. [Storage Systems](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/storage.md)
- Physical organization and performance of Hard Disk Drives (HDDs) and Solid State Drives (SSDs)
- File system structure: directory structure, index structure (inodes), storage blocks, free space map, hard links, soft links, and buffer cache
- Case studies: File Allocation Table (FAT) and Berkeley Fast File System (FFS)
- Replication as a means for durability: erasure codes and RAID (disk replication)
- Transactions as a means for reliability: journaling file systems and redo logging


### VII. [Networked and Distributed Systems](https://github.com/lynnliu030/os-prelim/blob/main/list_of_topics/network_ds.md)
- Communication: abstractions to use for communication
    - Remote Procedure Calls (RPCs) and their implementation
- Communication: Basic Messaging Layer
    - Reliable transport (TCP vs. UDP)
- Security in distributed system
    - Conceptual understanding of symmetric-key encryption, public-key encryption, and cryptographic hash functions (you need to know what these primitives are and how to use them, but formal security definitions are out of scope)
