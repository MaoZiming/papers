# Memory Management 
- Software support for address translation: page fault handler and multi-level page tables
- Hardware support for address translation: cache, TLB, and MMU
- Demand paging eviction policies: MIN (Belady's Algorithm), LRU (Least Recently Used), NRU (Not Recently Used, also known as the "Clock Algorithm"), and VAX/VMS second-chance lists
- Spatial locality, temporal locality, working sets, Belady's anomaly, and thrashing

## Mechanism: Address Translation

Main Goal - *Efficiency:* hardware support to perform translation quickly for each access

### HW support: Base-and-bound

- Store two HW registers in CPU’s MMU per process: base and bound registers
- When user program is running, HW translate each address
    - Physical addr = virtual addr + base
- Cons: *internal fragmentation* (i.e. space between stack and heap)

### Space mgmt #1: segmentation

- Segmentation allow OS to
    - place each one of those segments in different parts of physical memory
    - avoid filling physical memory with unused virtual address space
- Have a base-and-bound pair registers per logical segment (i.e. generalize base-bound)
    - Segments: code, stack, heap
    - Save and restore these register upon context switch
- Lookup
    - Address translation to segment number and offset
    - Use corresponding base+bound register for access
- Pros: fast, enable code sharing, better support for sparse addr space
- Cons: external fragmentation
    - I.e. different size of segments per process, physical memory full of little holes of free space, make it hard to allocate new segments
    - Solution
        - Compact physical memory by re-arranging existing segments
        - Free-list management: keep large extents of memory available for allocation
            - E.x. best-fit, worst-fit, first-fit, buddy algorithm

### Space mgmt #2: paging

- Physical memory is an array of fixed-sized slots called **page frames**
    - Use **page table**: per-process data structure from virtual → physical page
        - E.x. address = VPN + offset (within the page) —> PFN + offset
        - A page table entry indexed by VPN contains PFN +
            - valid, protection, present (disk / memory), dirty, reference bit
        - Page table register (PTBR): contain physical address of starting loc of PT
- Performance Optimization with **TLB (HW)**
    - Part of chip’s MMU, cache virtual-to-physical address translation
        - `VPN | PFN | other bit`
    - Handle TLB miss: HW/SW-managed TLB
        - HW talks PT v.s HW traps, OS trap handler
    - Issues
        - Per process: TLB flush on context switch (i.e. set valid bit = 0)
            - Sol: HW support of address space identifier (ASID) to enable sharing
        - Solution: replacement algorithm to minimize cache miss
    - Access memory with paging
        - Extract VPN and offset from virtual address
        - Check if TLB holds, if so, return; if not, TLB miss
            - Form address of page table entry: PTE = PTBR + (VPN * sizeof(PTE))
            - Check valid and protect bit, then fetch: PhysAddr = PFN + offset
- Space optimization with **page table design**
    - Linear page table too big (i.e. per process, multi-process)
    - Sol #1: bigger page? —> internal fragmentation
    - Sol #2: **page + segments**
        - Problem: most of the page table unused, full of invalid entries (e.x. between stack and heap)
        - Approach: one page table per logical segment
            - E.x. code, heap, stack
            - Have base and bound registers in MMU
                - Base: holds physical address of page table
                - Bound: holds the end of page table
        - TLB miss
            - Uses segment bit (SN) to determine which base and bound to use
            - PTE = Base[SN] + (VPN * sizeof(PTE))
            - Then using PTE to find PhysAddr
        - Cons
            - Still requires us to do segmentation (e.x. large but sparsely-used heap waste)
            - External fragmentation (i.e. page tables can be arbitrary size)
            - Finding free space more complicated                
    - Sol #3: **multi-level page table**
        - Turn linear page table to something like a tree
            - Chop up the page table into **page-sized units**
            - If an entire page of page-table entries (PTEs) is invalid, don’t allocate that page of the page table at all
            - Use **page directory**: track whether a page of the page table is valid
                - Either tell you where a page of the page table is
                - Or that entire page of the page table contains no valid page
        - The page directory, in a simple two-level table, contains one entry per page of the page table
            - Page directory entries (PDE)
                - Has a **valid bit** and a **page frame number**
                - Valid bit: if PDE is valid, it means that at least one of the pages of the page table that entry points to (via the PFN) is valid
        - Pros
            - Only allocate page-table space in proportion to the amount of addr space you are using, generally compact
            - If carefully constructed, each portion of the page table fits neatly within a page, making it easier to manage memory
                - V.s. linear page table, page table reside contiguously in physical memory
                - Here
                    - Add level of indirection through page directory, which points to pieces of page table
                    - Allow us to place page-table pages wherever we like in physical memory
        - Cons
            - On TLB miss, two loads from memory will be required to get the right translation info from page table
                - One to page directory, one to PTE
                - V.s. linear page table: one load
                - **Time-space trade-off**
                    - I.e. we want smaller tables, but not for free
                    - TLB miss suffers from higher cost with smaller table
            - Complexity
                - Page-table lookups more complicated in order to save valuable memory

<img width="622" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/f5a68fa3-1234-4859-813f-b94ae5f25e45">


### Other objectives

- *Transparent*: to process that has been relocated
- *Protection*: base-and-bounds, OS + hardware combine to ensure protection
    - E.x. segment register value with protection bits

## Paging

- Mapping information is generally stored in physical memory, paging logically requires extra memory lookup for each virtual address
- How can we **speed up address translation** and generally avoid the extra memory reference that paging seems to require?
- Also we need notion of accessing more memory than is physical present in the system transparently, how can we do that?

### Hardware Support: cache, TLB, MMU

- **What hardware support is required?**
    - ***Memory management unit (MMU)***
        - The part of processor that helps with address translation
        - For example, base-and-bound registers are stored here, TLB are stored here
    - ***Translation-lookaside buffer (TLB)***
        - Part of the chip’s  MMU
        - Simply a hardware cache of popular virtual-to-physical address translation
        - Maintain virtual-to-physical translations that are only valid for the current running process
            - Flush on context switches
        - Can be software-managed or hardware-managed
            - Software-managed: HW raises exception on TLB miss, jumps to kernel trap handler
    - ***Cache*** (TLB as a cache)
        - Cache replacement to minimize miss rate (i.e. LRU)
        - **Temporal locality**: when data is accessed, it is likely to be accessed again in near future
        - **Spacial locality**: if the program access a data item at address `x`, it is likely to access data items near `x` as well
    - **CPU cache**
        - L1, L2, L3 cache
        - L1: fastest, store data frequently accessed by CPU to speed up processing
        - L2: larger than L1 but slower, used to store data doesn’t fit in L1
        - L3: larger and slower than L2
        - Write through: write go through the cache to the main memory
        - Write back: write only update the cache, and main memory is updated later

### Software Support: page-fault handler and multi-level page table

- ***Multi-level page tables**  (look above)*
- ***Page fault handler***
    - **Page fault:** hen the HW looks in the PTE, it may find that page is *not present* in physical memory
        - Upon page fault, **OS is invoked with page-fault handler**
            - Find the page, do I/O from swap space (i.e. reserved space on the disk for moving pages back and forth)
            - Update page table to mark page as present, record in memory location of the new-fetched page, retry
    - The way that HW or OS determine this is through the present bit
        - 1: page is present in physical memory
        - 0: not in memory
    - If memory is full: **page replacement policy**

## Page Replacement Policy

### Cache misses

- Compulsory miss: cache is empty to begin with
- Capacity miss: cache run out of space, have to evict an item to bring new item
- Conflict miss: arise in hardware, not in OS page cache

### Page Replacement

| Policies  | Explain  |
| --- | --- |
| MIN (Belady's Algorithm)  | Throw away the pages that is needed furthest away from now. Guaranteed to minimize # of page faults.  |
| LRU (Least Recently Used) | Replace page not used for longest time in past | 
| NRU (Clock) | Approximate LRU with efficient implementation | 
| VAX/VMS second-chance lists | Second-chance list before complete eviction | 
| FIFO | Replace page that has been in memory the longest | 

### LRU
* Use past to predict future: replace pages not used for longest time in past
* Pros: locality, LRU approximate OPT
* Cons: hard to implement (must track which pages have been accessed), does not handle all workloads well

### NRU
* Approximate LRU with efficient implementation
* Clock algorithm: replace page that is “old enough”
* Hardware
  *  Keep use (or reference) bit for each page frame
  *  When page is referenced: set use bit
* Operating System (page replacement)
  - Keep pointer to last examined page frame
  - Traverse pages in circular buffer
  - Clear use bits as we search 
  - Stop when find page with already cleared use bit, replace this page

### VAX / VMS second-chance list 
Problem addressed: 
1) no reference bit
2) memory hogs can happen (i.e. programs use a lot of memory and make it hard for other programs to run)

Key features (approximate LRU): 
1. Resident set size (RSS): each process allocated a fixed # of page frames, pages are managed in FIFO 

2. Two second-chance list: before completely evicting a page from memory, it is put into two second chance list 

- 1) clean-page free list: hold pages that have not been modified since they were last loaded 
- 2) dirty-page list: modified 

When a process exceeds RSS:
1. the oldest page is taken from process’s FIFO list
2. if clean, placed at clean-page list, if dirty, go to dirty-page list 

When a process need to add a new page 
1. first look at free page from clean list
2. if process faults on a page that was moved to one of the second-chance list, it can reclaim it, avoid disk access

### FIFO
Replace page that has been in memory the longest. 

– Intuition: First referenced long time ago, done with it now
– Pros: All pages receive equal residency; Easy to implement
– Cons: Some pages may always be needed

### Add more physical memory, what happens to performance?
- LRU, OPT
    - Guaranteed to have fewer (or same number of) page faults
    - Smaller memory sizes are guaranteed to contain a subset of larger memory sizes
    - Stack property: smaller cache always subset of bigger
- FIFO
    - Usually have fewer page faults
    - ***Belady’s anomaly:*** May actually have more page faults!
      
### Other VM policies

- **Demand paging**: OS brings the page into memory when it is accessed
- **Prefetching**: bring it ahead of time

### Thrashing

- When memory is simply oversubscribed, and the memory demands of the set of running processes simply exceeds the available physical memory
- **The system will be constantly paging**: thrashing
- **Working set**: the pages that they are using actively
- Mechanisms to detect and cope
    - E.x. **Admission control:** not run a subset of the process, and hope that the working set fit in memory
    - E.x. **Out-of-memory killer**: run this when memory is oversubscribed, the daemon choose a memory-intensive process and kill it
