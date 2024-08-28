# Virtualization 
Virtualization allows multiple guest OSes (and their applications) to share the hardware resources of a single physical server. 

## Hosted vs. Hypervisor architecture

* Virtualization approaches use either a hosted or a hypervisor architecture. 
  * A **hosted architecture** installs and runs the virtualization layer as an application on top of an operating system and supports the broadest range of hardware configurations
  * In contrast, a **hypervisor** (bare-metal) architecture installs the virtualization layer directly on a clean x86-based system. Since it has direct access to the hardware resources rather than going through an operating system, a hypervisor is more efficient than a hosted architecture and delivers greater scalability, robustness and performance.

## Bare metal servers

* Bare metal servers are physical servers that are dedicated entirely to a single tenant or customer. Unlike virtualized environments where multiple virtual machines (VMs) share the same physical hardware, bare metal servers offer full access to all of the server's resources. 
* When you get a VM on AWS, it is not a bare metal server. Instead, it is a virtualized instance running on top of AWS's physical infrastructure, which is shared among many customers.
* AWS does offer bare metal instances (such as the i3.metal, m5.metal, and r5.metal instance types) where you have direct access to the physical hardware without a hypervisor.

## Challenges: trap-and-emulate 
* Guest instructions executed directly by hardware
    * Non-privileged operations: hardware speed
    * Privileged operations: **trap to hypervisor**
* Hypervisor determines what needs to be done
    * **Emulates** the behavior the guest OS expects from the hardware
    * Followed by "Return from Trap" (`rtt`)

* Commodity hardware has more than 2 protection levels. E.g. **X86 has 2 protection levels (rings) and 2 protection modes (root and non-root) recently.**

For X86 pre 2005, if trap-and-emulate: 
* 4 rings, hypervisor in ring 0, guest OS in ring 1 
* Problem: certain privileged instructions that can only be executed in Ring 0
   *  **17 privileged instructions do not trap, fail silently!**
   * E.x. enable / disable interrupt bit in privileged register from ring 1 fail silently 
* Hypervisor does not know, will not emulate the behaviors, assume changes was successful 

## CPU Virtualization

![alt text](images/71-virtualization/virtualization-comparison.png)

### Full virtualization: binary translation and direct execution
*  Goal: full virtualization == guest OS not modified
*  **Main idea: rewrite VM binrary to never issue those 17 instructions**
*  Approach: dynamic binary translation
    1) hypervisor inspects code to be executed by the guest OS 
    2) translates kernel code to replace nonvirtualizable instructions with new sequences of instructions that have the intended effect on the virtual hardware.
    3) if needed, translate to alternative instruction sequence
       * e.g. to emulate desired behavior, possibly even avoiding trap
    4) otherwise, run at hardware speeds
       * cache translated blocks to amortize translation costs 
*  Pros: best isolation and security, excellent compatibility
*  **Cons: performance** (trap to hypervisor)

### Paravirtualization: hypercalls
*  Goal: performance, give up on unmodified guest
*  Replace non-virtualizable instructions with **hypercalls**
*  The open-source Xen project is an example of paravirtualization. 
*  Approach: **paravirtualization** == **modify guest OS so that**
     * **it knows it's running virtualized**
     * **it makes explicit calls to the hypervisor (hypercalls)**
*  Hypercalls (~system calls)
     * package context info
     * specify desired hypercall
     * trap to VMM 
*  Pros: lower virtualization overhead
*  Cons: poor compatibility, requires OS kernel modifications 

### Hardware-assisted virtualization: root / non-root  
*  New hardwares (e.g. Intel VT-x and AMD's AMD-V) allows VMM to run in new root mode below ring 0
   *  **Privileged calls are set to automatically trap to hypervisor**
   *  Removing the need for either binary translation or paravirtualization. 
   *  E.x. Intel-VT (~2005) 
*  Pros: excellent compatibility
*  Cons: some performance issues
*  Rigid programming model: leaves little room for software flexibility in managing either the frequency or the cost of hypervisor to guest transitions. 

## Memory Virtualization 

* ![memory-virtualization](images/71-virtualization/memory-virtualization.png)

### Full virtualization 
The guests expect contiguous physical memory, start at 0 (just like it owns the machine). We should distinguish virtual v.s physical v.s machine addresses and page frame numbers, and still leverages hardware MMU and TLB. 
* Binary translation. the Host program would scan the guest binary for such instructions and replace them with either a call to hypervisor or some dummy opcode (which will cause a trap). 
  * In the case of para-virtualization, the source code has already been modified. Such an image directly calls hypervisor APIs. In the case of Binary translation, the native OS must first scan the guest OS instruction stream and make modifications to the stream as needed. Thus between the two, Para-Virtualization incurs lower overhead. 

**Option 1:**: expensive on every single memory reference 
* guest page table: VA => PA
* hypervisor: PA => MA 

**Option 2:**
* guest page table: VA => PA
* hypervisor shadow PT: VA => MA
  * VMM uses TLB hardware to map the virtual memory directly to the machine memory to avoid the two levels of translation on every access. 
* hypervisor maintains consistency between guest PT and hypervisor shadow PT 
  * When the guest OS changes the virtual memory to physical memory mapping, the VMM updates the shadow page tables to enable a direct lookup.
  * e.g. invalidate on context switch

### Paravirtualization
The guest is aware of virtualization, and there is no longer strict requirement on contiguous physical memory starting at 0. 

**The guest can *explicitly* registers page tables with the hypervisor. Every update to the page table will cause a trap to the hypervisor, but we can do tricks like "batch" page table updates and issue a single hypercall. Other optimizations can be useful.**

## Device and I/O virtualization 
When we look at CPU and memory, there is a significant level of standardization of interface at ISA-level and less diversity. We don't care much about the lower differences about the hardware. 

For devices, there is higher diversity, and there is lack of specificaiton of device interface and behavior. There are three key models for device virtualization (prior to virtualization even exists). 

![alt text](images/71-virtualization/device-driver.png)

* Passthrough model: VMM-level driver configures device access permissions
     *  Pros
         *  VM provided with *exclusive* access to device
         *  VM can directly access the device (**VMM-bypass**)
     *  Cons
         *  device sharing difficult
         *  VMM must have exact type of device as what VM expects
         *  VM migration tricky (i.e. device-specific states)
* Hypervisor-direct model: VMM intercepts all device accesses
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
* Split-device driver model: device access control split between
     *  front-end driver in guest VM (device API)
     *  back-end driver in service VM (or host)
     *  modified guest drivers
         *  i.e. limited to **paravirtualized** guests
     *  Pros
         *  eliminate emulation overhead, explicitly tell what guest VM requires
         *  allow for better management of shared devices     

## List of Questions 
* What is virtualization? how are they used today?
   * how do you virtualize hardware (cpu, memory, device)?
   * what is the design goal of Xen? and how does it accomplish it? 
* What is container? how are they used today?
   * how do you implement containerization?
   * should we use centralized or decentralized schedulers to handle containers 
   * what are some problems associated with containers? 
* What are the trade-offs between virtulization and container?
* How does exokernel compared to hypervisors?

## Answer to these questions

## What is Virtualization?
- **Definition:** Virtualization is the creation of a virtual (rather than actual) version of something, such as a hardware platform, operating system, storage device, or network resources. This allows multiple operating systems and applications to run on the same physical hardware simultaneously.

### How are they used today?
- **Server Consolidation:** Reducing the number of physical servers by running multiple virtual machines (VMs) on a single server.
- **Testing and Development:** Creating isolated environments for software development and testing.
- **Cloud Computing:** Providing scalable, on-demand resources in cloud environments.
- **Security:** Isolating applications to enhance security through compartmentalization.

### How do you virtualize hardware (CPU, memory, device)?
- **CPU:** Virtualization software (hypervisor) manages CPU time, allowing each VM to think it has its own CPU. Techniques include time-slicing and binary translation.
- **Memory:** Physical memory is abstracted into virtual memory spaces for each VM. Memory management units (MMUs) help in mapping virtual to physical addresses.
- **Devices:** I/O devices are virtualized through emulation or paravirtualization, where devices appear as if they are physical to the guest OS.

## Xen: Design Goals and Implementation
- **Design Goal:** To provide a scalable, efficient virtualization solution with minimal overhead.
- **How it Accomplishes This:**
  - **Microkernel Design:** Xen uses a microkernel approach where the hypervisor is minimal, providing only the necessary services for virtualization.
  - **Paravirtualization:** Allows guest OS to be modified for direct hypercalls, reducing the need for full hardware emulation.
  - **Hardware Virtualization Support:** Leverages CPU virtualization extensions for better performance when available.

## What is a Container?
- **Definition:** Containers are lightweight, standalone, executable software packages that include everything needed to run an application: code, runtime, system tools, system libraries, settings, etc.

### How are they used today?
- **Microservices Architecture:** Running each service in its own container for scalability and isolation.
- **Continuous Integration/Continuous Deployment (CI/CD):** For consistent environments from development to production.
- **Cloud Native Applications:** Designed for cloud environments, leveraging container orchestration tools like Kubernetes.

### How do you implement containerization?
- **Operating System Level Virtualization:** Containers share the host OS kernel, reducing overhead compared to full virtualization.
- **Container Runtimes:** Tools like Docker, rkt, or containerd manage the lifecycle of containers.
- **Isolation:** **Uses namespaces for process isolation and cgroups for resource control.**

### Centralized vs. Decentralized Schedulers for Containers
- **Centralized:** Easier management, but can become a bottleneck or single point of failure.
- **Decentralized:** More scalable, but can be complex to manage and coordinate.

### Problems Associated with Containers
- **Security:** Shared kernel can lead to vulnerabilities if not properly isolated.
- **Resource Management:** Containers can consume host resources unpredictably if not properly configured.
- **Networking:** Complex networking setups can be challenging to manage.

## Trade-offs Between Virtualization and Containerization
- **Isolation:** VMs offer stronger isolation since **each has its own OS**, whereas containers **share the host OS kernel**.
- **Performance:** Containers are generally faster to start and use fewer resources due to shared kernel.
- **Portability:** Containers are more portable across different environments due to their lightweight nature.
- **Complexity:** VMs require more setup and maintenance but are simpler in terms of isolation and security.

## Exokernel vs. Hypervisors
- **Exokernel:** 
  - **Goal:** To provide minimal abstraction between hardware and applications, allowing applications to manage hardware resources directly.
  - **Advantage:** Potentially higher performance due to less abstraction layers.
  - **Challenge:** Requires applications to be aware of and manage hardware details, increasing complexity.

- **Hypervisors:**
  - **Goal:** To create a layer of abstraction that allows multiple operating systems to run on a single hardware platform.
  - **Advantage:** Simplifies application development by abstracting hardware details, providing strong isolation.
  - **Challenge:** Adds overhead due to the abstraction layer, which can impact performance.

Both approaches aim to optimize resource usage and performance but differ significantly in their design philosophy regarding abstraction and control over hardware resources.

## Containers 
* Containers are lightweight, encapsulated environments that run applications and their dependencies.
* Unlike virtual machines, **containers share the host system’s kernel**, rather than needing their own operating system.
* Mechanisms: kernel featurers like **namespace** and **cgroups** 
    *  **Namespacing**: isolates process trees, networking user IDs, and mounts
    *  **Cgroups**: limit and prioritize CPU, memory, block I/Os, and network resources
*  Used in microservices
*  Pros: lightweight, fast, and portable 
*  Cons: **weaker isolation and security guarantees**

## Virtualization v.s Containers
* VM: virtualize hardware
* Containers: virtualize OS, contains only the application and its libraries and dependencies
* LightVM paper discusses the trade-offs between containers and VMs in terms of performance and security (isolation) guarantees

## Isolation and Fault tolerance 
It is important for fault tolerance because:
* Prevent a single point of failure from bringing down the entire system
* Limit the spread of security vulnerabilities
* Make it easier to identify and fix issues with modules

For example, UNIX provides isolation:
* Process isolation: each process runs in its own address space
* User and group permissions: file and process access control based on user and group permissions
* Namespace: used to isolate different system resources like network, PID, and mounts
   *  i.e. containers  

## Takeaways

* Full Virtualization with Binary Translation is the Most Established Technology Today
* Hardware Assist is the Future of Virtualization, but the Real Gains Have Yet to Arrive
* Xen’s CPU Paravirtualization Delivers Performance Benefits with Maintenance Costs


## Type I Virtualization vs. Type II Virtualization

* Type 1 hypervisor
  * A type 1 hypervisor, or a bare metal hypervisor, interacts directly with the underlying machine hardware. A bare metal hypervisor is installed directly on the host machine’s physical hardware, not through an operating system. In some cases, a type 1 hypervisor is embedded in the machine’s firmware.
  * The type 1 hypervisor negotiates directly with server hardware to allocate dedicated resources to VMs. It can also flexibly share resources, depending on various VM requests.
* Type 2 hypervisor
  * A type 2 hypervisor, or hosted hypervisor, interacts with the underlying host machine hardware through the host machine’s operating system. You install it on the machine, where it runs as an application.
  * The type 2 hypervisor negotiates with the operating system to obtain underlying system resources. However, the host operating system prioritizes its own functions and applications over the virtual workloads.

## VMWare ESX's Full Virtualization
* This approach uses hardware-assisted virtualization (like VT-x or AMD-V) or binary translation if hardware support isn't available.
* Performance: While modern hardware has reduced the performance gap, full virtualization initially had overhead due to the need for emulation or translation of certain instructions. However, with hardware support, it's now very efficient.

## How does Paravirtualization reduce latency.

### 1. **Reduced Overhead of Virtualization:**
   - **Direct Calls:** In traditional virtualization, guest OS operations often go through a trap to the hypervisor, which then emulates the hardware. Paravirtualization allows the guest OS to make direct calls to the hypervisor for certain operations, bypassing the need for emulation. This direct communication reduces the overhead associated with context switches and hardware emulation.

### 2. **Optimized System Calls:**
   - **Modified Guest OS:** Paravirtualization involves modifying the guest OS to be aware of the hypervisor. This means system calls that would normally require complex hardware emulation can be replaced with more efficient hypercalls. These hypercalls are specifically designed for the virtualized environment, reducing the number of steps and thus the latency.

### 3. **Efficient I/O Handling:**
   - **Paravirtualized Drivers:** Traditional virtualized environments use emulated hardware, which can be slow. Paravirtualized drivers are designed to work directly with the hypervisor's I/O model, allowing for more efficient data transfer. For instance, network and disk I/O can be significantly faster because they use optimized paths rather than emulating physical devices.

### 4. **Reduced Context Switches:**
   - **Less Trapping:** In a paravirtualized environment, the guest OS knows it's running on a hypervisor, so it can avoid unnecessary traps to the hypervisor for operations that can be handled more efficiently. This reduces the number of context switches, which are inherently latency-inducing.

## Trap in full virtualization vs. Hypercall in paravirtualization

## Hypercalls in Paravirtualization

**Definition:**
- A hypercall is a direct call from a guest operating system (OS) to the hypervisor in a paravirtualized environment.

**Advantages:**
- **Reduced Latency:** Since there's no need for the guest OS to emulate hardware, operations are faster.
- **Efficiency:** Fewer layers of abstraction mean less overhead.

**Example:**
- In Xen paravirtualization, the guest OS might use hypercalls for memory management or I/O operations, directly interacting with Xen's hypervisor.

## Traps into the Hypervisor in Full Virtualization

**Definition:**
- A trap into the hypervisor occurs when the guest OS, unaware of being virtualized, attempts to perform an operation that requires privileged access or hardware interaction, triggering a trap to the hypervisor.

**Mechanism:**
- **Emulation:** The hypervisor intercepts these operations, emulates the hardware behavior, and then returns control to the guest OS.
- **Context Switch:** Each trap involves a context switch from the guest to the hypervisor and back, which incurs overhead.

**Advantages:**
- **Compatibility:** No modification to the guest OS is required, allowing any OS to run without changes.
- **Isolation:** Provides a high level of isolation as the guest OS believes it's running on real hardware.

**Disadvantages:**
- **Increased Latency:** The emulation process adds significant overhead due to the need for context switches and hardware emulation.
- **Performance Impact:** Full virtualization can be less efficient due to these extra steps.

**Example:**
- In VMware or KVM, when a guest OS tries to access hardware directly (like changing a page table), it triggers a trap, and the hypervisor handles the request by emulating the hardware response.

## Summary Comparison

- **Hypercalls (Paravirtualization):**
  - **Speed:** Faster due to direct calls.
  - **Guest OS:** Requires modification to be hypervisor-aware.
  - **Use Case:** Best for environments where guest OS can be modified for performance.

- **Traps (Full Virtualization):**
  - **Speed:** Slower due to emulation and context switching.
  - **Guest OS:** No modification needed, any OS can run.
  - **Use Case:** Ideal for running unmodified guest OS or when compatibility is paramount over performance.

## TLB Virtualization

Virtualizing the Translation Lookaside Buffer (TLB) is crucial for maintaining efficient memory management in virtualized environments. Below is an explanation of how TLB virtualization is handled in various contexts:

## 1. Shadow Page Tables
- **Concept:** Hypervisors use **shadow page tables** to manage TLB entries in early virtualization techniques.
- **Mechanism:** 
  - The hypervisor maintains page tables that map guest virtual addresses (GVAs) directly to host physical addresses (HPAs).
  - When the guest OS modifies its page tables (GVA to GPA), the hypervisor intercepts and updates the shadow page tables.
  - The TLB caches these shadow page table entries to allow direct translation from GVA to HPA.
- **Performance Impact:** 
  - This approach can be slow because every guest page table update must be intercepted by the hypervisor.

## 2. Hardware-Assisted TLB Virtualization (Nested/Extended Page Tables)
- Modern CPUs provide hardware-assisted memory virtualization:
  - **Nested Page Tables (NPT)** in AMD processors.
  - **Extended Page Tables (EPT)** in Intel processors.
- **Two-Level Translation:**
  1. Guest OS handles GVA → GPA translation.
  2. Hypervisor handles GPA → HPA translation.
- **TLB Management:**
  - The TLB can cache two-level translations (GVA → HPA) directly.
  - The hardware manages both page table levels, reducing the need for hypervisor intervention.
- **TLB Flushes:** Hardware-assisted virtualization minimizes full TLB flushes by using tagged TLBs (discussed below).

## 3. Tagged TLB (Process-Context Identifiers - PCIDs)
- **Concept:** Tagged TLBs associate entries with unique identifiers (e.g., process IDs or VM IDs).
- **Usage in Virtualization:** 
  - Allows storing TLB entries from multiple address spaces simultaneously.
  - Reduces the need for full TLB flushes during context switches, improving performance.

## 4. TLB Shootdowns in Virtualization
- **TLB Shootdowns:** When page tables are modified, corresponding TLB entries need to be invalidated.
- **Handling in Hypervisors:** The hypervisor may intercept TLB shootdown requests from the guest OS and coordinate with hardware to invalidate the correct TLB entries across all cores and threads.

## 5. Emulation and Paravirtualization
- **Emulated TLB Management:** 
  - Hypervisors may need to emulate TLB operations for privileged instructions related to memory management.
  - This can be slower but is necessary in fully virtualized environments.
- **Paravirtualized TLB Management:**
  - In paravirtualized environments (e.g., Xen's PV mode), the guest OS interacts directly with the hypervisor for TLB management.
  - The guest OS explicitly notifies the hypervisor when TLB entries need to be flushed or modified, improving efficiency.


## Process Migration

* Typically move the process and leave some support for it back on the original machine
* E.g., old host handles local disk access, forwards network traffic
* these are “residual dependencies” – old host must remain up and in use

## VM Migration

VM migration typically involves transferring a running virtual machine (VM) from one physical host to another. The process can be broken down into different phases:

### 1. Push Phase
In the **push phase**, the memory pages of the VM are copied from the source machine to the destination machine while the VM is still running. This phase is also referred to as **pre-copy migration**. During this phase:

- The memory pages are iteratively copied to the destination.
- Pages that are modified (dirtied) during the copying process are tracked and re-copied in the next iteration.
- The goal is to reduce the amount of data that needs to be transferred during the actual switchover.

The push phase allows the VM to continue running with minimal disruption, but the migration process can take longer if the VM's memory is being modified frequently.

### 2. Stop-and-Copy Phase
The **stop-and-copy phase** involves stopping the VM on the source host, copying any remaining dirty memory pages to the destination, and then resuming the VM on the destination host. In this phase:

- The VM is temporarily halted to ensure no further memory changes occur while the final set of pages is transferred.
- The stop-and-copy phase is usually very brief to minimize downtime.
- Once the transfer is complete, the VM starts running on the destination machine, and the migration process finishes.

This phase is critical for consistency, as it ensures that the VM's memory state is fully synchronized between the source and destination. However, this phase introduces downtime, as the VM must be paused.

### 3. Pull Phase
In the **pull phase**, the VM is started on the destination machine, but it may still require some memory pages that are not yet available locally. During this phase:

- When the VM tries to access a page that hasn't been transferred, it generates a page fault.
- The destination host requests the missing page from the source host, which sends it over the network.
- This continues until all necessary pages are present on the destination host.

The pull phase reduces downtime because the VM can start running on the destination before all memory pages are transferred. However, it can degrade performance initially, as the VM may experience frequent page faults and network delays while retrieving missing pages.

### Summary
- **Push phase**: Memory pages are transferred while the VM continues running.
- **Stop-and-copy phase**: The VM is stopped, remaining memory pages are copied, and the VM is resumed on the destination host (brief downtime).
- **Pull phase**: The VM starts running on the destination, and missing memory pages are fetched as needed (potential performance impact initially).
