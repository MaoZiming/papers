# OS-prelim

Book: https://pages.cs.wisc.edu/~remzi/OSTEP/

Notion: https://www.notion.so/Intro-a5776b0729184768b1c4263da2f8095e

This is a repo consisted of paper-reading notes for Berkeley OS prelim exam

Sorted By Topics

* Concurrency and CPU Scheduling
* Communication: Local and Remote
* Storage Systems
* Memory Management
* Distributed System Theory
* Protection and Security
* System Structure
* Application Structure and Programming Models
* Cluster Computing
* Bugs and Correctness
* Mobile, Ubiquitous, and Edge Computing
* Revealed Truth

This is a repo consisted of my paper-reading notes for Berkeley OS prelim exam
* Prelim [reading list](https://ucbosprelim.samkumar.org/reading.html)
* Basic OS materials (🙌 to Remzi and Andrea): [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/)
* Practice exams
    *  Wisconsin [OS Quals](https://www.cs.wisc.edu/operating-systems-quals/)
    *  Berkeley past prelim [questions](https://www2.eecs.berkeley.edu/Protected/Grads/CS/Prelims/osqu.html)
* Super useful references from people taken it before 
    *  Paras's [repo](https://github.com/parasj/papers)
    *  Peter's paper [reading note](https://pschafhalter.com/blog) 

## Basic Overview 
Here are a [list of resources](https://ucbosprelim.samkumar.org/undergraduate.html) to review undergrad materials for OS. 

Summary on all these concepts are [here](https://github.com/lynnliu030/os-prelim/tree/main/list_of_topics), including OS concepts and abstractions, synchronization, CPU scheduling, memory management, I/O and system performance, storage system, and network and distributed systems. 

## Sorted By Topics 
* [Concurrency and CPU Scheduling](https://github.com/lynnliu030/os-prelim/tree/main/cpu_concurrency)
* [Communication: Local and Remote](https://github.com/lynnliu030/os-prelim/tree/main/communication)
* [Storage Systems](https://github.com/lynnliu030/os-prelim/tree/main/storage)
* [Memory Management](https://github.com/lynnliu030/os-prelim/tree/main/memory)
* [Distributed System Theory](https://github.com/lynnliu030/os-prelim/tree/main/theory)
* [Protection and Security](https://github.com/lynnliu030/os-prelim/tree/main/security)
* [System Structure](https://github.com/lynnliu030/os-prelim/tree/main/system_structure)
* [Application Structure and Programming Models](https://github.com/lynnliu030/os-prelim/tree/main/programming_model)
* [Cluster Computing](https://github.com/lynnliu030/os-prelim/tree/main/cluster_computing)
* [Bugs and Correctness](https://github.com/lynnliu030/os-prelim/tree/main/security)
* [Mobile, Ubiquitous, and Edge Computing](https://github.com/lynnliu030/os-prelim/tree/main/edge)
* [Revealed Truth](https://github.com/lynnliu030/os-prelim/tree/main/reveal_truth)

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
