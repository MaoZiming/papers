# Virtualization 
Virtualization allows multiple guest OSes (and their applications) to share the hardware resources of a single physical server. 

Reference: https://learn.udacity.com/courses/ud923/lessons/1df60659-fbbb-4dee-85fc-6c70af48f3dc/concepts/39d66818-78e5-4a49-b382-7aafc9b6c202

## Challenges: trap-and-emulate 
* Guest instructions executed directly by hardware
    * Non-privileged operations: hardware speed
    * Privileged operations: trap to hypervisor
* Hypervisor determines what needs to be done
    * Emulates the behavior the guest OS expects from the hardware

Commodity hardware has more than 2 protection levels. E.g. X86 has 2 protection levels (rings) and 2 protection modes (root and non-root) recently. 

For X86 pre 2005, if trap-and-emulate: 
* 4 rings, hypervisor in ring 0, guest OS in ring 1 
* Problem: certain privileged instructions that can only be executed in Ring 0
   *  17 privileged instructions do not trap, fail silently! 
   * E.x. enable / disable interrupt bit in privileged register, done through POPF / PUSHF instructions that access it from ring 1 fail silently 
* Hypervisor does not know, will not emulate the behaviors, assume changes was successful 

## CPU Virtualization

<img width="890" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/45d1a8e1-ca3f-4feb-904e-31b40f94d483">


### Full virtualization: binary translation and direct execution
*  Goal: full virtualization == guest OS not modified
*  Main idea: rewrite VM binrary to never issue those 17 instructions 
*  Approach: dynamic binary translation
    1) hypervisor inspect code to be executed by the guest OS 
    2) if needed, translate to alternate instruction sequence
       - e.g. to emulate desired behavior, possibly even avoiding trap
    3) otherwise, run at hardware speeds
       - cache translated blocks to amortize translation costs 
*  Pros: best isolation and security, excellent compatibility
*  Cons: performance 
### Paravirtualization: hypercalls
*  Goal: performance, give up on unmodified guest
*  Approach: paravirtualization == modify guest OS so that
     - it knows it's running virtualized
     - it makes explicit calls to the hypervisor (hypercalls)
*  Hypercalls (~system calls)
     - package context info
     - specify desired hypercall
     - trap to VMM 
*  Pros: lower virtualization overhead
*  Cons: poor compatibility, requires OS kernel modifications 

### Hardware-assisted virtualization: root / non-root  
*  Allow VMM to run in new root mode below ring 0
   *  Privileged calls are set to automatically trap to hypervisor
   *  E.x. Intel-VT (~2005) 
*  Pros: excellent compatibility
*  Cons: some performance issues
   


## Memory Virtualization 

<img width="426" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/07f5f4b2-ff73-4834-913f-30373f6ef170">

### Full virtualization 
The guests expect contiguous physical memory, start at 0 (just like it owns the machine). We should distinguish virtual v.s physical v.s machine addresses and page frame numbers, and still leverages hardware MMU and TLB. 

Option 1: expensive on every single memory reference 
- guest page table: VA => PA
- hypervisor: PA => MA 

**Option 2:**
- guest page table: VA => PA
- hypervisor shadow PT: VA => MA
- hypervisor maintains consistence between guest PT and hypervisor shadow PT 
    - e.g. invalidate on context switch, write-protect guest PT to track new 

### Paravirtualization
The guest is aware of virtualization, and there is no longer strict requirement on contiguous physical memory starting at 0. 

The guest can explicitly registers page tables with the hypervisor. Every update to the page table will cause a trap to the hypervisor, but we can do tricks like "batch" page table updates and issue a single hypercall. Other optimizations can be useful.

## Device and I/O virtualization 
When we look at CPU and memory, there is a significant level of standardization of interface at ISA-level and less diversity. We don't care much about the lower differences about the hardware. 

For devices, there is higher diversity, and there is lack of specificaiton of device interface and behavior (e.x. what is the behavior when a particular call is invoked on a device). There are three key models for device virtualization (prior to virtualization even exists). 

<img width="865" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/91f86049-bd5c-4e69-a938-adf1d36552f3">


1. Passthrough model: VMM-level driver configures device access permissions
     *  Pros
         *  VM provided with exclusive access to device
         *  VM can directly access the device (VMM-bypass)
     *  Cons
         *  device sharing difficult
         *  VMM must have exact type of device as what VM expects
         *  VM migration tricky (i.e. device-specific states)
2. Hypervisor-direct model: VMM intercepts all device accesses
     *   Emulate device operation
           *  translate generic I/O operation
           *  traverse VMM-resident I/O stack
           *  invoke VMM-resident driver
     *   Pros
         *  VM decoupled from physical device
         *  sharing, migration, dealing with device specifics
     *   Cons
         *  latency of device operations
         *  device driver ecosystem complexities in hypervisor
3. Split-device driver model: device access control split between
     *  front-end driver in guest VM (device API)
     *  back-end driver in service VM (or host)
     *  modified guest drivers
         *  i.e. limited to paravirtualized guests
     *  Pros
         *  eliminate emulation overhead, explicitly tell what guest VM requires
         *  allow for better management of shared devices     
