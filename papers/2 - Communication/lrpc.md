# Lightweight Remote Procedure Call (LRPC) 

Link: https://lass.cs.umass.edu/~shenoy/courses/spring04/677/readings/Bershad_lightwt_rpc.pdf

Read: June 17th, 2024.

This paper presents a lightweight communication facility (based on RPC) designed to **handle communication where calling process and the called process are on the same machine**. Through a profiling based approach, they identified that **most RPC was local and small size**. 

If both processes reside on the same machine, original RPC involves redundant data copy and operations, including network stacks, marshaling / unmarshaling, XDR, etc. Thus they optimize for this case to increase performance. 

Cross-domain calls are implemented with facilities required by cross-machine ones. 

By microkernel.

## Main Ideas 
- No need for marshaling in this situation
- Eliminate the need for explicit message passing. Rather memory is employed as means of communication.
- The stub can utilize the run-time flag to determine if TCP/IP or shared memory should be used.
- Not necessary to use external data representation (XDR)
  
A server module exports an interface through a clerk in the LRPC run-time library included in every domain. The clerk registers the interface with a name server and awaits import requests from clients. A client binds to a specific interface by making an import call via the kernel. The importer waits while the kernel notifies the server’s waiting clerk.

The clerk enables the binding by replying to the kernel with a procedure descriptor list (PDL) that is maintained by the exporter of every interface. The PDL contains one procedure descriptor (PD) for each procedure in the interface. The PD includes an entry address in the server domain, the number of simulta- neous calls initially permitted to the procedure by the client, and the size of the procedure’s argument stack (A-stack) on which arguments and return values will be placed during a call. For each PD, the kernel pairwise allocates in the client and server domains a number of A-stacks equal to the number of simultaneous calls allowed. These A-stacks are mapped read-write and shared by both domains.

After the binding has completed, the kernel returns to the client a Binding Object. The Binding Object is the client’s key for accessing the server’s interface and must be presented to the kernel at each call. The kernel can detect a forged Binding Object, so clients cannot bypass the binding phase. In addition to the Binding Object, the client receives an A-stack list for each procedure in the interface giving the size and location of the A-stacks that should be used for calls into that procedure.



1. **Simple control transfer**
    1. When client initiates the call, the kernel verifies the client and transfers control directly to the server’s domain to execute the required procedure 
    2. When it is done, control returned back to client 
2. **Simple data transfer**
    1. Use shared argument stack to avoid duplicative data copying 
3. **Simple stubs** 
    1. “Stubs”: code for setting up remote call and handling data transfer 
    2. Generation of highly optimized stubs 
4. **Design for concurrency** 
    1. Idle cores can be used to cache  

## Steps of LRPC
1. **Kernel trap**: client calls procedure
    1. arguments of the calling process are pushed on the stack
    2. argument stack is writable by both client and server 
2. **Shared memory creation**: after sending trap to kernel (in kernel mode), it either
    1. constructs an explicit shared memory region and put the arguments there
    2. or take the page from stack and simply turn it into shared page 
5. **Procedure execute**d by the client thread in the server’s domain (**OS upcall**)
    1. when client’s thread triggers this procedure, hands off control to the server code 
    2. Upcall is performed **into the server's stub** at the address specified in the PD for the registered procedure. 
6. **Return to Client**
    1. after the procedure is done, another trap to the kernel
    2. kernel again changes back the address space and returns control to the client
  
## v.s. upcalls 
Both papers aim to optimize IPC but do so by tackling different problems and proposing different solutions. Upcalls focuses on the vertical integration within a system across different layers, while LRPC focuses on the horizontal interaction between processes, especially on the same machine. Both papers touch on the idea of using a single address space for optimization. 

## Summary

This paper presents a lightweight communication facility (based on RPC) designed to handle communication where **calling process and the called process are on the same machine**. 

## Key Insights

### Notes: communication of distributed system

1. Explicit: passing data 
    1. Process communicate by message passing on the heterogeneous platforms over a network 
    2. Handles cross-machine traffic 
2. Implicit: sharing memory 
    1. Memory cannot be shared physically, rather shared memory is distributed that permits the end-user process to access shared data without IPC 
    2. It handles cross-domain (domains on the same machine) traffic 

Rather than statically allocating E-stacks, LRPC delays the A-stack/E-stack association until it is needed, that is, until a call is made with an A-stack not having an associated E-stack. When this happens, the kernel checks if there is an E-stack already allocated in the server domain, but currently unassociated with any A-stack. If so, the kernel associates the E-stack with the A-stack. Otherwise, the kernal allocates an E-stack out of the server domain and associates it with the A-stack. 

a simple LRPC needs only one formal procedure call (into the client stub) and two returns (one out of the server procedure and one out of the client stub).

LRPC reduces context-switch overhead by caching domains on idle processors.

### Problem

If both processes are using shared memory location and reside on the same machine, In standard RPC, your package will go down OS and the IP layer will realize they are process on the same machine. It will come back up and call another process. 

- Involves the network stacks, marshaling / unmarshaling, XDR, etc.



### Insights

The following considerations help it outperform standard RPC approach in the same setup: 

- No need for marshaling in this situation
- Eliminate the need for explicit message passing. Rather memory is employed as means of communication.
- The stub can utilize the run-time flag to determine if TCP/IP or shared memory should be used.
- Not necessary to use external data representation (XDR)

## Key Techniques

### Steps of LRPC execution

1. **Arguments** of the calling process are pushed **on the stack**
2. **Trap to kernel** is send
3. **Shared memory creation**: after sending trap to kernel (in kernel mode), it either
    1. constructs an explicit shared memory region and put the arguments there
    2. or take the page from stack and simply turn it into shared page 
4. **Procedure execute**d by the client thread in the server’s domain (**OS upcall**)
    1. When client’s thread triggers this procedure, hands off control to the server code 
5. **Return to Client**
    1. After the procedure is done, another trap to the kernel
    2. Kernel again changes back the address space and returns control to the client 

### Four main points

1. **Simple control transfer**
    1. “Handoff scheduling” to minimize delay and resource usage 
    2. When client initiates the call, the kernel verifies the client and transfers control directly to the server’s domain to execute the required procedure 
    3. When it is done, control returned back to client 
2. **Simple data transfer**
    1. Use shared argument stack to avoid duplicative data copying 
3. **Simple stubs** 
    1. “Stubs”: code for setting up remote call and handling data transfer 
    2. Each domain has its own stub, reduce overhead costs of layer transitions in communication 
4. **Design for concurrency** 
    1. Maximize tput and minimize latency through shared memory and multi-processors 
    2. Reduce unnecessary lock contention and use shared data structures

Deciding whether a call is cross-domain or cross-machine is made at the earliest possible moment: the first instruction of the stub. If the call is to a truly remote server (indicated by a bit in the Binding Object), then a branch is taken to a more conventional RPC stub. The extra level of indirection is negligible compared to the overheads that are part of even the most efficient network RPC implementation.

The stub can utilize the run-time flag to determine if TCP/IP or shared memory should be used.

in LRPC the copying of the same arguments can be only made once, from the client’s stack to the shared argument stack. 

The highly optimized stubs can be easily generated using LRPC because control and data transfer mechanisms are employed.  The client’s domain is having a call stub and the server’s domain is having an entry stub in every procedure. 

