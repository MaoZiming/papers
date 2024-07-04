# Xen and the Art of Virtualization (2003) 
This paper proposes a practical design for paravirtualized virtual machine. The main goal of the design is performance. 

## Motivation: inefficiency of full virtualization 
To support correct virtualization, certain supervisor instructions must be handled by the hypervisor (VMM), but executing this with insufficient privilege fails silently rather than causing a trap. Full virtualization techniques (e.x. VMWare's ESX) dynamically rewrites portion of the guest machine code to insert traps whenever VMM intervention is required. 

This technique is **complex** and has **high performance cost** particularly for update-intensive operations (i.e. trap every update to maintain consistency with virtual tables). Also, sometimes it is desirable for guest OS to see real resources: i.e. real and virtual time, expose real machine addresses to improve performance. 

## Interface 
1. Memory 
   *  guests have read access to PT but must update them through VMM 
   *  can do batching of updates 
2. CPU
   *  replaces nonvirtualizable instructions with **hypercalls** that communicate directly with the VMM
   *  guest OS uses Ring 1, Xen uses Ring 0
   *  guest registers trap vector table with VMM
   *  system calls utilize handlers for faster operation 
   *  guests are aware of virtual and real time
3. I/O
   *  data transfer between Xen and guest OS is through sync I/O rings 

## Problem 
Bad portability and maintenance cost because of the need to modify guest OS. 
