# Lightweight Remote Procedure Call (LRPC) 

Link: https://lass.cs.umass.edu/~shenoy/courses/spring04/677/readings/Bershad_lightwt_rpc.pdf

Read: June 17th, 2024.

- This paper presents a lightweight communication facility (based on RPC) designed to **handle communication where calling process and the called process are on the same machine**. Through a profiling based approach, they identified that **most RPC was local and small size**. 
- If both processes reside on the same machine, original **RPC** involves redundant data copy and operations, including network stacks, marshaling / unmarshaling, XDR, etc. Thus they optimize for this case to increase performance. 
  - In standard RPC, your package will go down OS and the IP layer will realize they are process on the same machine. It will come back up and call another process. 
- Explicit: passing data 
    - Process communicate by message passing on the heterogeneous platforms over a network 
    - Handles cross-machine traffic 
- Implicit: sharing memory 
    - Memory cannot be shared physically, rather shared memory is distributed that permits the end-user process to access shared data without IPC 
    - It handles cross-domain (domains on the same machine) traffic 

## Main Ideas 
- No need for marshaling on the same machine.
- Eliminate the need for explicit message passing. Rather memory is employed as means of communication.
- The stub can utilize the run-time flag to determine if TCP/IP or shared memory should be used.
- Not necessary to use external data representation (XDR, data serialization format).

## Steps of LRPC
1. **Kernel trap**: client calls procedure
    1. arguments of the calling process are pushed on the stack
    2. argument stack is writable by both client and server 
2. **Shared memory creation**: after sending trap to kernel (in kernel mode), it either
    1. constructs an explicit shared memory region and put the arguments there
    2. or take the page from stack and simply turn it into shared page 
3. **Procedure executed** by the client thread in the server’s domain (**OS upcall**)
    1. when client’s thread triggers this procedure, hands off control to the server code 
    2. Upcall is performed **into the server's stub** at the address specified in the PD (Process descriptor) for the registered procedure. 
4. **Return to Client**
    1. after the procedure is done, another trap to the kernel
    2. kernel again changes back the address space and returns control to the client

### Four main points

1. **Simple control transfer**
    1. “Handoff scheduling” to minimize delay and resource usage 
    2. When client initiates the call, the kernel verifies the client and transfers control directly to the server’s domain to execute the required procedure 
    3. When it is done, control returned back to client 
2. **Simple data transfer**
    1. Use **shared argument stack** to avoid duplicative data copying 
    2. Only one copying: from client stack to the shared argument stack. 
3. **Simple stubs** 
    1. *Stubs*: code for setting up remote call and handling data transfer 
    2. Each domain has its own stub, reduce overhead costs of layer transitions in communication.
    3. The client’s domain is having a call stub and the server’s domain is having an entry stub in every procedure. 
4. **Design for concurrency** 
    1. Maximize tput and minimize latency through shared memory and multi-processors 
    2. Reduce unnecessary lock contention and use shared data structures

### RPC Explain 

Read: June 15 2024. 

1. **Local Processing** (In User Space)
**Marshalling**: The client program initiates the RPC by calling the stub procedure, passing the necessary parameters. The client stub then **marshals (packages) these parameters** into a format suitable for transmission over a network.
1. **Communication to Kernel Space**
* System Call: After marshalling, a system call is made to pass the data from user space to kernel space.
* Data is copied from the user space memory to the kernel space memory.
1. **Transmission Over Network** (From Kernel Space)
* Network IO: The kernel then transmits this data packet over the network to the remote system.
1. **Reception at Remote System** (In Kernel Space)
* Data packet is first received in the kernel space of the remote system.
1. **Communication to User Space at Remote System**
* system Call: Through a system call, the data packet is moved from kernel space to user space, involving another data copy operation.
1. **Remote Processing** (In User Space)
    * **Unmarshalling**: the server stub unmarshals (unpacks) the parameters from the data packet.
    * **Execution**: The actual procedure on the server-side is then executed using the unpacked parameters.
2. **Sending the Response**
  * Marshalling the Response: The server stub marshals the result into a data packet.
  * System Call to Kernel Space: The data packet is passed to the kernel space through a system call, involving a data copy operation.
  * Transmission Over Network: The kernel sends this data packet over the network to the client system.
3. **Receiving the Response** at Client
  * Reception in Kernel Space: The data packet with the result reaches the kernel space of the client system.
  * System Call to User Space: Through a system call, the data packet is moved to the user space, involving a data copy operation.
4. **Final Unmarshalling** (In User Space)

### Data Copy 
1. Copy from client user space to client kernel space (while sending the request).
2. Copy from server kernel space to server user space (to process the request).
3. Copy from server user space to server kernel space (to send the response).
4. Copy from client kernel space to client user space (to unmarshal the response).

### Message passing 
Whenever a piece of data is to be exchanged, the process will invoke the kernel with a request to send information to target process. 

User-mode process will copy data into buffer, then issue a system call to request the data to be transferred. Once kernel is invoked, it will copy the transferred data first to its own memory. Then target process do similar things. 

E.x. pipe, message queue, RPC 

**Cons**: 
1) one data exchange invokes two syscalls, one read, one write
2) transferred data are copied twice (i.e. kernel memory, receiving process). 

### Shared memory 
Processes initially set up a region of their virtual memory to use for IPC. Then it issues a system call to request that the kernel make the region shared. Other processes would do the same. The processes can read from and write to the region as normal, and there is no explicit syscall required to read new data. 

**Pros**: one-time penalty during set-up phase 

**Cons**: synchronization overhead 