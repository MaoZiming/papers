# Exokernel: An Operating System Architecture for Application-Level Resource Management (1995) 

Link: https://pdos.csail.mit.edu/6.828/2008/readings/engler95exokernel.pdf

Read: June 29th, 2024. 

Exokernels are an attempt to separate security from abstraction, making non-overrideable parts of the operating system do next to nothing but securely multiplex the hardware.

**In the exokernel architecture, a small kernel securely exports all hardware resources through a low- level interface to untrusted library operating systems.**


Substantial evidence exists that applications can benefit greatly from having more control over how machine resources are used to implement higher-level abstractions.

- Virtual machines have severe performance penalities. 
it exports hardware resources rather than emu- lating them, which allows an efficient and simple implementation.
- Applications can securely bind to machine resources and handle events. 
- Applications participate in resource revocation protocol. 
- Exokernel can break the binding of uncooperative applications by force. 

Virtual memory and inter-process communication abstractions are implemented entirely within an application-level library. 
- virtual memory is implemented at application level, where it can be tightly integrated with distributed shared memory systems and garbage collectors.

The goal is to avoid forcing any particular abstraction upon applications, instead **allowing them to use or implement whatever abstractions** are best suited to their task without having to layer them on top of other abstractions which may impose limits or unnecessary overhead. 


Fixed high-level abstractions hurt application performance be- cause there is no single way to abstract physical resources or to implement an abstraction that is best for all applications.

This is done by **moving abstractions into untrusted user-space libraries** called "library operating systems" (libOSes), which are linked to applications and call the operating system on their behalf.

page-table structures can vary among library operating systems: an application can select a library with a particular implementation of a page table that is most suitable to its needs.

Kernel cross can be smaller, since most of the operating system runs in the address space of the application. 

A programming error in one library OS should not affect another library operating system. 

An exokernel should avoid resource management. It should only manage resources to the extent required by protection (i.e., management of allocation, revocation, and ownership).

> *Eliminating the notion that an operating system must provide abstractions upon which to build applications*

However, as in all systems, an ex- okernel must include policy to arbitrate between competing library operating systems: it must determine the absolute importance of different applications, their share of resources, etc. This situation is no different than in traditional kernels.

By isolating the need to understand these semantics to bind time, the kernel can efficiently implement access checks at access time without under- standing them. Simply put, a secure binding allows the kernel to protect resources without understanding them.

Another example is the packet filter [37], which allows predicates to be downloaded into the kernel (bind time) and then run on every incoming packet to de- termine which application the packet is for (access time).

Secure bindings can be cached in say, software TLBs. 

For example, with the exception of some external pagers [2, 43], most operating systems deallocate (and allocate) physical memory without informing applications. This form of revocation has lower latency than visible revocation since it requires no application involvement. Its disadvantages are that library operating systems cannot guide deallocation and have no knowledge that resources are scarce.

An exokernel uses visible revocation for most resources. 

If a library operating system fails to comply with the revocation protocol, an exokernel simply breaks all existing secure bindings to the resource and informs the library operating system.

To record the forced loss of a resource, we use a repossession vector. When an exokernel takes a resource from a library oper- ating system, this fact is registered in the vector and the library operating system receives a “repossession” exception so that it can update any mappings that use the resource. For resources with state, an exokernel can write the state into another memory or disk resource. In preparation, the library operating system can pre-load the repossession vector with a list of resources that can be used for this purpose.

Another complication is that an exokernel should not arbitrarily choose the resource to repossess. A library operating system may use some physical memory to store vital bootstrap information such as exception handlers and page tables. The simplest way to deal with this is to guarantee each library operating system a small number of resources that will not be repossessed (e.g., five to ten physical memory pages)

## Design Principals
### Main Insight: E2E
- Applications know better than OS what the goal of their resource management decisions should be
- They should be given as much control as possible those decisions

### Principals 

1. Separate protection from mgmt (policy): resource management is restricted to functions necessary for protection.
2. Expose allocation: applications allocate resources explicitly.
3. Expose name: use physical names wherever possible, access hardware directly 
4. Expose revocation: let applications to choose which instance of a resource to give up
5. Expose information: expose all system information and collect data that applications cannot easily derive locally

## Pros and Cons
Pros 
1. Significant performance improvement for applications 
2. Applications can make more efficient and intelligent use of hardware resources by being aware of resource availability, revocation and allocation.
3. Ease development and testing of new operating system ideas. (New scheduling techniques, memory management methods, etc)

Cons 
1. Complexity in design of exokernel interfaces.
2. Less consistency.

### Others
Today’s trends is that hardware is becoming faster, this might make exokernel design principals more valuable 
- E.x DB system that needs high performance often completely forgo FS all together and manage raw block
- DPDK in Linux: let application bypass kernel abstraction (i.e. storage, networking stack) and go directly to the raw hardware
