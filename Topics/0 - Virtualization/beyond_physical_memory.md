# Beyond Physical Memory: Mechanisms

- Notion of accessing more memory than is physically present within the system
- Mechanisms: more complexity in page-table structure
    - **present bit**
    - **page-fault handler** in OS to service the page fault
    - Upon page fault, **OS is invoked with page-fault handler**
        - Find the page, do I/O from swap space (i.e. reserved space on the disk for moving pages back and forth)
        - Update page table to mark page as present, record in memory location of the new-fetched page, retry
- All of these actions take place ***transparently*** to the process
    - pages are placed in arbitrary (non-contiguous) locations in physical memory, and sometimes they are not even present in memory, requiring a fetch from disk
- Before, we assume that address space is unrealistically small and fits into physical memory (i.e. every address space of every running process)
- Now we want to relax those assumptions
    - For *convenience* and *ease-of-use*
    - V.s. memory overlays: programmers manually move pieces of code or data in and out of memory
- Require an additional level in the **memory hierarchy**
    - Usually use hard disk drive

- **Swap Space**: reserved space on the disk for moving pages back and forth
    - Assume size of swap space is very large
- The first thing we will need to do is to reserve some space on the disk for moving pages back and forth. In operating systems, we generally refer to such space as swap space, because we swap pages out of memory to it and swap pages into memory from it.
  - Thus, we will simply assume that the OS can read from and write to the swap space, in page-sized units. To do so, the OS will need to remember the disk address of a given page.

## Page Fault Control Flow 

- OS must find a physical frame for the soon-to-be-faulted-in page to reside within
    - If no such page, wait for replacement algorithm to free
- With a physical frame in hand, handler issues I/O request to read in page from swap space
- Finally, when the slow operation completes, OS updates the page table and retries the instruction
  - Retry will result in TLB miss
  - Then another retry, a TLB hit, where HW will be able to access the desired item 
- How it works
    - When OS notices that there are fewer than LW pages available
    - Background thread (i.e. **swap daemon** or **page daemon**) responsible for freeing memory runs
    - Thread evicts pages until there are HW pages available
- Remember that the hardware first extracts the VPN from the virtual address, checks the TLB for a match (a TLB hit), and if a hit, produces the resulting physical address and fetches it from memory. This is hopefully the common case, as it is fast (requiring no additional memory accesses).
If the VPN is not found in the TLB (i.e., a TLB miss), the hardware locates the page table in memory (using the page table base register) and looks up the page table entry (PTE) for this page using the VPN as an index. If the page is valid and present in physical memory, the hardware extracts the PFN from the PTE, installs it in the TLB, and retries the instruction, this time generating a TLB hit; so far, so good.
- the OS will then update the page table to mark the page as present, update the PFN field of the page-table entry (PTE) to record the in-memory location of the newly-fetched page, and retry the instruction. This next attempt may generate a TLB miss, which would then be serviced and update the TLB with the translation (one could alternately update the TLB when servicing the page fault to avoid this step). Finally, a last restart would find the translation in the TLB and thus proceed to fetch the desired data or instruction from memory at the translated physical address.
- many systems will cluster or group a number of pages and write them out at once to the swap parti- tion, thus increasing the efficiency of the disk