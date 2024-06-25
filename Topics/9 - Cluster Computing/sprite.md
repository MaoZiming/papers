# Transparent Process Migration: Design Alternatives and the Sprite Implementation (1991)  
The paper talks about design of process migration in the Sprite OS. The objective is to utilize idle workstations in a network by migrating processes from busy to idle machines, without disrupting user experience. The authors balance four key factors: **transparency**, residual dependencies, performance, and complexity.

Key techniques
1. Global State Maintenance: all machine in the same network see the same namespace
2. State Transfer: Virtual memory, open files, process IDs, etc., are transferred from the source to the target machine
3. Forward Kernel Calls: Certain kernel calls, like gettimeofday(), are forwarded to the home machine for execution

Examples
1. Migrating Virtual Memory: The paper uses a variant of lazy-copying the pages, flushing the dirty pages to the server first and then starting the process on the target with no resident pages.
2. Migrating Open Files: The paper transfers file references, caching information, and access positions between the source and target machines to maintain file state consistency.

## Compared with VMs or containers 
### Similarities 
* Goal: improve resource utilization, provide transparency 
* State transfer: migration involves transfer state like memory contents and file descriptors
    *  vm migration: entire memory state, CPU, and device state
    *  container migration: container's FS, process states, network connections
*  Lazy copy: modern hypervisors also deploy similar techniques
    *  only necessary parts of memory are transferred initially
    *  rest are paged / transferred on demand
*  Global namespace maintenance: Sprite's idea of single-system image is similar to container orchestration system like K8s, where there is an abstraction layer over individual node 

### Differences 
* Forward kernel call: not a standard practice due to better networking and API solutions 
* VM and Containers: more coarse-grained, moving the entire VM or container applications (often grouped into Pods in K9s), the goal is to manage and move them effectively as a unit
   * Why? consistency, performance, and isolation 
