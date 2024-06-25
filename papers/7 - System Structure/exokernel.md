# Exokernel: An Operating System Architecture for Application-Level Resource Management (1995) 
Exokernels are an attempt to separate security from abstraction, making non-overrideable parts of the operating system do next to nothing but securely multiplex the hardware.

The goal is to avoid forcing any particular abstraction upon applications, instead **allowing them to use or implement whatever abstractions** are best suited to their task without having to layer them on top of other abstractions which may impose limits or unnecessary overhead. 

This is done by **moving abstractions into untrusted user-space libraries** called "library operating systems" (libOSes), which are linked to applications and call the operating system on their behalf.

> *Eliminating the notion that an operating system must provide abstractions upon which to build applications*

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
