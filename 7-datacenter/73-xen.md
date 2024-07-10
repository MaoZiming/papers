# Xen and the Art of Virtualization (2003) 

Link: https://www.cl.cam.ac.uk/research/srg/netos/papers/2003-xensosp.pdf

Read: July 10th, 2024.

This paper proposes a practical design for **paravirtualized** virtual machine. The main goal of the design is performance. 

## Motivation: inefficiency of full virtualization 

> This paper presents Xen, an x86 virtual machine monitor which allows multiple commodity operating systems to share conventional hardware in a safe and resource managed fashion, but without sacrificing either performance or functionality. 

* Requirements
  * Isolation
  * Wide variety of OS
  * Performance penalty should be low. 

To support correct virtualization, **certain supervisor instructions must be handled by the hypervisor (VMM)**, but executing this with insufficient privilege fails silently rather than causing a trap. **Full virtualization** techniques (e.x. VMWare's ESX) dynamically rewrites portion of the guest machine code to insert traps whenever VMM intervention is required. For example, ESX implements shadow versions of system structure such as page tables and maintain consistency with the virtual tables by trapping every update attempt. 

This technique is **complex** and has **high performance cost** particularly for update-intensive operations (i.e. trap every update to maintain consistency with virtual tables). Also, sometimes it is desirable for guest OS to see real resources: i.e. real and virtual time, expose real machine addresses to improve performance. 

- There are situations in which it is desirable for the hosted operating systems to see real as well as virtual resources: providing both real and virtual time allows a guest OS to better support time-sensitive tasks, and to correctly handle TCP timeouts and RTT estimates. 

- In Denali, applications are linked explicitly against an instance of the Ilwaco guest OS in a manner rather reminiscent of a libOS in the Exokernel. Each virtual machine essentially hosts a single-user single-application un-protected “operating system”. In Xen, by contrast, a single virtual machine hosts a real operating system which may itself securely multiplex thousands of unmodified user-level processes.

## Interface 
1. Domain
   1. A running virtual machine within which a guest OS executes. 
2. Memory 
   *  guests have read access to PT but must update them through VMM. In other words, relinquish direct write privileges to the PT memory; all subsequent updates must be **validated** by Xen. A domain may be allocated discountiguous machine pages. 
   *  can do batching of updates 
   *  Challenge: x86 does not have software-managed TLB; instead, TLB misses are serviced automatically by the processor by walking the page table structure in hardware. 
   *  The top 64MB region of each address space, which is reserved for Xen, is not accessible or remappable by guest OSes. This avoids a TLB flush when entering and leaving the hypervisor. 
3. CPU
   *  replaces nonvirtualizable instructions with **hypercalls** that communicate directly with the VMM
   *  x86 has four protection rings, but typically only two are used.
   *  guest OS uses Ring 1, Xen uses Ring 0. Guest OS needs to run at lower priviledge levels. 
      *  Guest OS (on Ring 1) is prevented from directly executing privileged instructions.
   *  guest **registers a descriptor table for exception handling with VMM**. 
   *  guests are aware of virtual and real time
   *  Xen improves the performance of system calls by allowing **each guest OS to register a ‘fast’ exception handler** which is accessed directly by the processor without indirecting via ring 0 (i.e. such fast exception handler does not need to interact with Xen); this handler is validated before installing it in the hardware exception table. 
4. I/O
   *  I/O data is transferred to and from each domain (of the guest OS) via Xen, using shared-memory, asynchronous buffer- descriptor rings.
   *  Xen supports a lightweight event-delivery mechanism which is used for sending asynchronous notifications to a domain.
      *  **The event mechanism replaces hardware interrupt for notifications.**
   *  These notifications are made by updating a bitmap of pending event types and, optionally, by calling an event handler specified by the guest OS. 
      *  The callbacks can be 'held off' at the discretion of the guest OS - to avoid extra costs of frequent interruptions. 
*  ![alt text](images/73-xen/system-structure.png)

## Problem 
Bad portability and maintenance cost **because of the need to modify guest OS.**

Luckily, it does not require changes to the application binary interface. 
