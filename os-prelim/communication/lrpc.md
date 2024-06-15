# Lightweight Remote Procedure Call (LRPC) 
This paper presents a lightweight communication facility (based on RPC) designed to **handle communication where calling process and the called process are on the same machine**. Through a profiling based approach, they identified that **most RPC was local and small size**. 

If both processes reside on the same machine, original RPC involves redundant data copy and operations, including network stacks, marshaling / unmarshaling, XDR, etc. Thus they optimize for this case to increase performance. 

## Main Ideas 
- No need for marshaling in this situation
- Eliminate the need for explicit message passing. Rather memory is employed as means of communication.
- The stub can utilize the run-time flag to determine if TCP/IP or shared memory should be used.
- Not necessary to use external data representation (XDR)
  
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
6. **Return to Client**
    1. after the procedure is done, another trap to the kernel
    2. kernel again changes back the address space and returns control to the client
  
## v.s. upcalls 
Both papers aim to optimize IPC but do so by tackling different problems and proposing different solutions. Upcalls focuses on the vertical integration within a system across different layers, while LRPC focuses on the horizontal interaction between processes, especially on the same machine. Both papers touch on the idea of using a single address space for optimization. 





