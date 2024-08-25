# Lightweight Remote Procedure Call (LRPC) 

Link: https://lass.cs.umass.edu/~shenoy/courses/spring04/677/readings/Bershad_lightwt_rpc.pdf

Read: June 17th, 2024.

* This paper presents a lightweight communication facility (based on RPC) designed to **handle communication where calling process and the called process are on the same machine**. Through a profiling based approach, they identified that **most RPC was local and small size**. 
* If both processes reside on the same machine, original **RPC** involves redundant data copy and operations, including network stacks, marshaling / unmarshaling, XDR, etc. Thus they optimize for this case to increase performance. 
  * In standard RPC, your package will go down OS and the IP layer will realize they are process on the same machine. It will come back up and call another process. 
* Explicit: passing data 
    * Process communicate by message passing on the heterogeneous platforms over a network 
    * Handles cross-machine traffic 
* Implicit: sharing memory 
    * Memory cannot be shared physically, rather shared memory is distributed that permits the end-user process to access shared data without IPC 
    * It handles cross-domain (domains on the same machine) traffic 

* These observations indicate that simple byte copying is usually sufficient for transferring data across system interfaces and that the majority of interface procedures move only small amounts of data.

* In existing systems, cross-domain calls are implemented in terms of the facilities required by cross-machine ones.

## Main Ideas 
* No need for marshaling on the same machine.
* Eliminate the need for explicit message passing. Rather **memory** is employed as means of communication.
* The stub can utilize the **run-time flag** to **determine if TCP/IP or shared memory** should be used.
* Not necessary to use external data representation (XDR, data serialization format).

## Steps of LRPC
* **Kernel trap**: client calls procedure
    * arguments of the calling process are pushed on the stack
    * argument stack is writable by both client and server 
* **Shared memory creation**: after sending trap to kernel (in kernel mode), it either
    * constructs an explicit shared memory region and put the arguments there
    * or take the page from stack and simply turn it into shared page 
* **Procedure executed** by the client thread in the server’s domain (**OS upcall**)
    * when client’s thread triggers this procedure, hands off control to the server code 
    * Upcall is performed **into the server's stub** at the address specified in the PD (Process descriptor) for the registered procedure. 
* **Return to Client**
    * after the procedure is done, another trap to the kernel
    * kernel again changes back the address space and returns control to the client

### Four main points

* **Simple control transfer**
    * “Handoff scheduling” to minimize delay and resource usage 
    * When client initiates the call, the kernel verifies the client and transfers control directly to the server’s domain to execute the required procedure 
    * When it is done, control returned back to client 
* **Simple data transfer**
    * Use **shared argument stack** to avoid duplicative data copying 
    * Only one copying: from client stack to the shared argument stack. 
* **Simple stubs** 
    * *Stubs*: code for setting up remote call and handling data transfer 
    * Each domain has its own stub, reduce overhead costs of layer transitions in communication.
    * The client’s domain is having a call stub and the server’s domain is having an entry stub in every procedure. 
* **Design for concurrency** 
    * Maximize tput and minimize latency through shared memory and multi-processors 
    * Reduce unnecessary lock contention and use shared data structures

### RPC Explain 

Read: June 15 202* 

* **Local Processing** (In User Space)
**Marshalling**: The client program initiates the RPC by calling the stub procedure, passing the necessary parameters. The client stub then **marshals (packages) these parameters** into a format suitable for transmission over a network.
* **Communication to Kernel Space**
* System Call: After marshalling, a system call is made to pass the data from user space to kernel space.
* Data is copied from the user space memory to the kernel space memory.
* **Transmission Over Network** (From Kernel Space)
* Network IO: The kernel then transmits this data packet over the network to the remote system.
* **Reception at Remote System** (In Kernel Space)
* Data packet is first received in the kernel space of the remote system.
* **Communication to User Space at Remote System**
* system Call: Through a system call, the data packet is moved from kernel space to user space, involving another data copy operation.
* **Remote Processing** (In User Space)
    * **Unmarshalling**: the server stub unmarshals (unpacks) the parameters from the data packet.
    * **Execution**: The actual procedure on the server-side is then executed using the unpacked parameters.
* **Sending the Response**
  * Marshalling the Response: The server stub marshals the result into a data packet.
  * System Call to Kernel Space: The data packet is passed to the kernel space through a system call, involving a data copy operation.
  * Transmission Over Network: The kernel sends this data packet over the network to the client system.
* **Receiving the Response** at Client
  * Reception in Kernel Space: The data packet with the result reaches the kernel space of the client system.
  * System Call to User Space: Through a system call, the data packet is moved to the user space, involving a data copy operation.
* **Final Unmarshalling** (In User Space)

### Data Copy 
* Copy from client user space to client kernel space (while sending the request).
* Copy from server kernel space to server user space (to process the request).
* Copy from server user space to server kernel space (to send the response).
* Copy from client kernel space to client user space (to unmarshal the response).

### Message passing 
* Whenever a piece of data is to be exchanged, the process will invoke the kernel with a request to send information to target process. 

* User-mode process will copy data into buffer, then issue a system call to request the data to be transferred. Once kernel is invoked, it will copy the transferred data first to its own memory. Then target process do similar things. 

* E.x. pipe, message queue, RPC 

### A-Stack and E-stack

* When the client invokes a remote procedure, the call transfers control directly to the server, which starts executing on the client’s stack. The server's execution uses the same stack frames created by the client’s procedure call.
* After the server completes the procedure, control is returned back to the client, continuing from where the client left off, again using the shared stack.
* A-Stack: Argument stack
  * the kernel pairwise allocates in the client and server domains a number of A-stacks equal to the number of simultaneous calls allowed. These A-stacks are mapped read-write and shared by both domains.

* E-stack: Execution stack
* Rather than statically allocating E-stacks, LRPC delays the A-stack/E-stack association until it is needed, that is, until a call is made with an A-stack not having an associated E-stack. 
* When this happens, the kernel checks if there is an E-stack already allocated in the server domain, but currently unassociated with any A-stack. 
  * If so, the kernel associates the E-stack with the A-stack. 
  * Otherwise, the kernal allocates an E-stack out of the server domain and associates it with the A-stack. 
* When the call returns, the E-stack and A-stack remain associated with one another so that they might be used together soon for another call (A-stacks are LIFO managed by the client). 
* Whenever the supply of E-stacks for a given server domain runs low, the kernel reclaims those associated with A-stacks that have not been used recently.



**Cons**: 
1) one data exchange invokes two syscalls, one read, one write
2) transferred data are copied twice (i.e. kernel memory, receiving process). 

### Shared memory 
* Processes initially set up a region of their virtual memory to use for IPC. Then it issues a system call to request that the kernel make the region shared. Other processes would do the same. **The processes can read from and write to the region as normal, and there is no explicit syscall required to read new data.**

**Pros**: one-time penalty during set-up phase 

**Cons**: synchronization overhead 