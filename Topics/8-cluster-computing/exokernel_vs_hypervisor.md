# Sample Questions 
Question #1: Exokernels and Hypervisors
Exokernels and hypervisors have many similarities, in that they **multiplex and protect hardware with minimal abstraction**.
1. How does the exokernel manage address translations and memory reclamation (i.e., page replacement)?
2. How does a hypervisor like Disco or VMware manage address translations and memory reclamation?
3. What performance optimizations are available in one system (Exokernel or hypervisor) and not available in the other
system design? Explain your answer.
4. People have proposed running library operating systems in hypervisors. Suppose you want to run all the applications on
your system with their own library operating system. Would you be better off using an Exokernel or hypervisor for this
configuration? Explain your answer

## Exokernel 
1. Address Translations: Exokernels typically delegate the responsibility of address translation to applications. They use hardware-based mechanisms to ensure that applications cannot access unauthorized memory regions.
2. Memory Reclamation (Page Replacement): Exokernels tend to give individual applications the freedom to manage their own pages. Memory reclamation is generally offloaded to the application level or to specialized libraries, allowing for customized algorithms best suited for the application’s needs.

## Hypervisor 
1. Address Translations: Hypervisors use hardware-assisted or software-emulated MMUs to create isolated address spaces for each virtual machine (VM). This ensures each VM thinks it's using physical memory, while actually using a portion of the host's physical memory. (i.e. guest page table, then hypervisor page table). 
2. Memory Reclamation (Page Replacement): Hypervisors typically manage memory at the VM granularity, using techniques like ballooning or page sharing to reclaim memory from less active VMs.


## Comparisons (Q3) 
* Exokernels: They allow applications to make fine-grained decisions about resource management, which can lead to highly optimized usage of hardware. This is not typically possible in a hypervisor model where the hypervisor itself abstracts away much of the hardware details.
* Hypervisors: They benefit from optimizations like VM migration for load balancing, and memory deduplication across VMs, which isn't naturally a part of the Exokernel design, as exokernels don't manage multiple isolated OS instances.

## Use-cases 
Exokernel Advantage: If each application will manage its own resources closely with its own library OS, exokernels would offer better performance and customization opportunities. The exokernel is designed to offload as much policy as possible to the application or library OS level, thereby providing more flexibility.

Hypervisor Advantage: If isolation between different applications (running in their own library OS) is a major concern, then a hypervisor would be better suited because it's designed to fully isolate multiple guest operating systems.
