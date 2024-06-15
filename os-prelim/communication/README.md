# Communication 
Reference: https://github.com/parasj/papers/tree/master/communications

When we think about communication, we need to think about multiple layers. 

1. Local communication: IPC related concepts, shared memory v.s message passing
2. Remote communication
   *  how to build a reliable transport? (i.e. UDP v.s TCP)
   *  what abstractions of communication should we use when building a distributed system? (i.e. RPC) 
   *  how to make it efficient and fast? how to make it secure? 

**Upcall** and **LRPC** focus on local communication (different aspects, vertically and horizontally), and **active messages** focus on remote communication. These three papers are all primarily concerned with performance. 

## Communication [IPC]: message passing v.s shared memory 
Inter-process communication (IPC) is a mechanism that allows processes to communicate with each other and synchronize their actions. Processes can communicate with each other through either shared memory model or message passing model.

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

## RPC: an abstraction for communication  
What is RPC? 
* Just a PL abstraction, create wrappers so calling a function on another machine just feels like calling a local function.
* The _stub function_ removes the pain of packing function arguments and results into messages by automating it.
* This also systems to communicate across multiple languages. Protobuf is an example of a serialization/deserialization format across languages.

LRPC deals with making RPC lightweight. Active Messages offers an alternative model where message handlers are triggered to initiate computation, and demonstrates attention paid to optimize latency for small messages. These needs sometimes bypass the conventional networking stack. 

## Security v.s Performance 
Many RPC libraries make security tradeoffs to improve performance. For example, LRPC is where clients execute code directly in the server's domain. For upcalls, single address space makes things faster but also there is a risk on isolation and upward dependencies. 
