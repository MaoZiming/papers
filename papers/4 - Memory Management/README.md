# Virtual memory 
Overview: https://pages.cs.wisc.edu/~sschang/OS-Qual/memory/Memory.htm

Reference: https://github.com/parasj/papers/tree/master/virtual_memory 

## Space Management Mechanisms: Segmentation v.s Paging 
Segmentation and paging are two different techniques that OS uses for memory space management. 

A segment is a contiguous portion of address space of a particular length, e.x. code, stack, heap. What segmentation allows the OS to do is to place each one of those segments in different parts of physical memory, and thus avoid filling physical memory with unused virtual address space. This is accomplished by storing base-and-bound pair register per logical segment (e.x. code, stakc, heap) and use that for data access. During lookup, first we extract segment number and offset from virtual address, then use the corresponding base register for that segment + offset to access physical address. 

Paged memory is based on linear sequence of fixed-size pages. They are simple to allocte and manage. Each process has a separate page table where it stores the virtual page number to physical page number. It mitigates the problem of external fragmentation, but the page table needs to be carefully managed to speed up memory access time and save spaces. 

Ultimately, segmentation is better as a logical unit of memory (thus better for protection and sharing) while pages are better as a physical unit of information as they simplify implementation. Both approaches are complementary, as Multics attempted.

## HW support for VM: registers and TLB 
Segmentation is very simple to support in hardware with the base and bound registers stored in MMU. 

With paging mechanism, TLB is often used to provide better memory access performance as a HW cache of virtual-to-physical address translations. On context switch, the TLB is flushed. It is order 10x faster than main memory. 
