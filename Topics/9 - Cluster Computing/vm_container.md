# My VM is Lighter (and Safer) than your Container (2017)
The paper addresses the trade-off between isolation and efficiency in Virtual Machines (VMs) and containers. While containers are lightweight, they lack the security guarantees that VMs provide. The paper proposes techniques based on Xen to make VMs as efficient as containers without compromising on security.

## Tradeoffs 
Containers are lightweight but less secure, while VMs offer strong isolation at the expense of being resource-heavy and slow to boot.
*  Why is container less secured?
    *  shared kernel 
    *  containers use large # of syscall APIs to interact with HostOS, larger attack surface 

## Key technique 
Unikernels: create minimalistic VMs where a pared-down OS is directly 

1. Lightweight guest: TinyX 
   *  automate creations of trimmed-down Linux VMs just enough to run the applications
   *  take standard Linux Distro and make it smaller 
2. Re-architect toolstack
   *  re-architect Xen's toostack (i.e. control plane)
       *  get rid of Xenstore: store data, guest comm, sync
       *  use shared memory and event channel for communication     
   *  optimized to offer fast boot-times that scale to large # of VMs  

## Use cases 
JIT service instantiation in mobile edge computing, lightweight compute services like AWS Lambda 
