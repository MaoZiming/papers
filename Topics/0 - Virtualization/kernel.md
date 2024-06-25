# Kernel

- **Types of kernel**
    - **Monolithic kernel: speed!**
        - E.x. Unix, Linux, BSD, Multics
        - All OS services operate in kernel space
            - Provides scheduling, file management through sys calls
        - Pros
            - Performance: communication between components is fast and efficient!
        - Cons
            - Complexity: huge
            - Poor maintainability: hard to debug and maintain
            - Poor reliability: if any service fails, system failure

- **Microkernel**: **modularity and reliability**
    - E.x. L4
    - Minimalistic approach
        - Kernel only keep: basic IPC, virtual memory, scheduling
        - Put other services run in user space
            - Libraries running on user space
            - E.x. device drivers, networking, file system, etc.
        - Communication between components is provided by IPC (i.e. applications R/W through IPC)
    - Pros
        - Scalability: kernel smaller, separation of services at separate layers
        - Extensibility: new services can be easily added
        - Easy maintenance and debugging
        - More secure and reliable: a single service faults not lead to failures
    - Cons
        - Bad performance: lots of system calls (i.e. IPC) and context switches
    - File read —> communicate with the file system service residing in the user space through IPC. This involves a series of message exchanges and possibly more context switches

- **Hybrid kernel**
    - E.x. Windows, Android
    - Combine best of both worlds
        - Speed and simple design of a monolithic kernel
        - Modularity and stability of a microkernel
    - Pros: still similar to monolithic kernel
        - With disadvantages similar

- **Exokernel**
    - Follow end-to-end principals
        - Extremely minimal: extreme version of microkernel
        - Fewest hardware abstractions as possible
        - Just allocates physical resources to apps
    - Pros: minimal and simple
    - Cons
        - More work for application developer
        - Poor isolation: each application implements own LibOS
            - Libraries accessing directly the HW, don’t have isolations
        - Hardware compatibility: need to change LibOS depends on HW interfaces

- **Unikernel** = Exokernel + Containers
    - Goal: link application with just enough OS functionality to allow it to execute
    - Key idea: run one application per VM, one process per application
        - Everything compiled into a VM image
    - Pros
        - extremely lightweight (~MB), tailored to perform just one specific application
        - customizable
        - better isolation
    - Cons
        - Recompile, each libraries untrusted

### 1.2 Hardware Privilege Level

- Setting managed by the CPU to restrict types of operations that can be executed
- X86: four privilege levels
    - **Ring 0:** highest privilege level, generally used for most trusted functions of OS kernel
    - **Ring 1, Ring 2:** seldom use
    - **Ring 3:** lowest privilege levels, typically used for user-mode
- Connections
    - Kernel mode: typically high hardware privilege level (e.g., Ring 0)
    - User mode: typically low hardware privilege level (e.g., Ring 3).