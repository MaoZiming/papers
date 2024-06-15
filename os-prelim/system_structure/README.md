# System Structure 
## Different types of kernel 
<img width="1280" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/325fc7dc-2bd8-466f-838b-2bac23719d7f">

The kernel is a computer program at the core of a computer's operating system and generally has complete control over everything in the system. Its responsibilities include disk, memory, CPU, and device management, and it provides an interface between user and hardware components of a system. 

### Monolithic Kernel 
Examples of this include Unix, Linux, BSD, and Multics. All OS services operate in kernel space, OS provides scheduling, file management through sys calls. 
* _Pros_: Performance - communication between components is fast and efficient!
* _Cons_
   *  Complexity: huge kernel
   *  Poor maintainability: hard to debug and maintain
   *  Poor reliability: if any service fails, system failure 
   

### Microkernel
Examples of this include L4. Microkernel takes a minimalistic approach: kernel space only keep the basic functionalities (e.x. basic IPC, virtual memory, scheduling), and put other services to run in user space as libraries (e.x. device drivers, networking, file system, etc.). Communication between components is provided by IPC. 

- _Pros_
    - Scalability: kernel smaller, separation of services at separate layers
    - Extensibility: new services can be easily added
    - Easy maintenance and debugging
    - More secure and reliable: a single service faults not lead to failures
- _Cons_
    - Bad performance: lots of system calls and context switches

### Hybrid Kernel
Examples of this include Windows and Android. The goal is to combine best of both worlds, to provide the speed and simple design of a monolithic kernel, and the modularity and stability of a microkernel

- _Pros_: still similar to monolithic kernel
    - With disadvantages similar
   
### Exokernel
Exokernel is an extreme of microkernel, which follow an end-to-end principals: the kernel is extremely minimal and provide fewest hardware abstractions as possible (i.e. just allocate physical resources to applications).

- _Pros_: minimal and simple
- _Cons_
    - More work for application developer
    - Poor isolation: each application implements own LibOS
        - Libraries accessing directly the HW, don’t have isolations
    - Hardware compatibility: need to change LibOS depends on HW interfaces

### Unikernel 
Unikernel is frequently used in the cloud setup, which is basically an exokernel with containers. The goal of Unikernel was to link applicaiton with just enough OS functionality to allow it to execute. The key idea is that you run one application per VM, and one process per application, and everything compiled to a VM image.

- _Pros_
    - extremely lightweight (~MB), tailored to perform just one specific application
    - customizable
- _Cons_
    - Similar to exokernels 

## Design Tradeoffs 
* Import functionality into kernel vs expose hardware?
* Cost of abstrations
* Monolithic kernels v.s microkernels 

## Flexibility v.s Reliability v.s Security 
