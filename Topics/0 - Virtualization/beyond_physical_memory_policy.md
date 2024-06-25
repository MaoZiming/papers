# Beyond Physical Memory: Policies

- Page-replacement policy
    - Scan-resistant algorithms are usually LRU-like but also try to avoid the worst-case behavior of LRU, which we saw with the looping-sequential workload
- Types of cache misses:
    - **Code-start miss** (**compulsory miss**): cache is empty to begin with, first reference to the item
    - **Capacity miss**: cache run out of space and had to evict an item to bring a new item
    - **Conflict miss**: arise in hardware because of limits on where an item can be placed in a hardware cache, due to set-associativity
        - It does not arise in OS page cache because caches are always fully-associative (i.e. there are no restrictions on where in memory a page can be placed)
- FIFO: pages were placed in a queue when they enter the system
    - If cache gets larger, with FIFO, it gets worse! (Belady’s Anomaly)
    - Might kick out important pages
- LRU: least recently used
    - Smaller memory sizes are guaranteed to contain a subset of larger memory sizes
    - Stack property: smaller cache always subset of bigger
    - Looping-sequential workload: perform bad
- Approximate LRU
    - Require some hardware support (i.e. use bit or reference bit)
    - Clock algorithm
        - All pages arranged in a circular list
        - A clock hand points to some particular page to begin with
        - When replacement must occur, OS checks the currently-pointed to page P has a use bit of 1 or 0
            - If 1: P recently used, not good candidate, then its bit to 0, increment to next page
            - Continues to find a page of
        - Nice property: not repeatedly scanning through all of memory to look for an unused page
    - Modification to clock: consider whether a page has been modified or not while in memory
        - If it has been modified and is thus dirty, must be written back to disk to evict it, which is expensive
        - If not modified (and is thus clean), the eviction is free
        - Physical frame can simply be reused for other purposes without additional I/O
        - Thus some VM systems prefer to evict clean pages over dirty pages
        - So
            - HW includes a **dirty bit**, set when a page is written
- Other VM policy
    - OS decides when to bring a page into memory (i.e. page selection policy)
        - Most pages: demand paging, bring in on-demand
        - Prefetching: i.e. page P is brought into memory, code P + 1 will likely soon be accessed
    - OS decides how to write pages out to disk
        - One at a time v.s clustering (or grouping)

### Thrashing
- Question: what should the OS do when memory is simply oversubscribed, and the memory demands of the set of running processes simply exceeds the available physical memory?
- In this case, the system will constantly be paging —> **thrashing**
- Approach: both detect and cope
    - E.x. **Admission control:** not run a subset of the process, and hope that the working set fit in memory
    - E.x. **Out-of-memory killer**: run this when memory is oversubscribed, the daemon choose a memory-intensive process and kill it


### Page Replacement. 

Policies 	Explain 
MIN (Belady's Algorithm) 	Throw away the pages that is needed furthest away from now. Guaranteed to minimize # of page faults. 
LRU (Least Recently Used)	Least-recently-used: Replace page not used for longest time in past

– Intuition: Use past to predict the future
– Advantages: With locality, LRU approximates OPT
– Disadvantages:
• Harder to implement, must track which pages have been accessed
• Does not handle all workloads well
NRU (Not Recently Used, also known as the "Clock Algorithm")	Approximate LRU with efficient implementation 

Clock algorithm: replace page that is “old enough” 

Hardware
– Keep use (or reference) bit for each page frame
– When page is referenced: set use bit

Operating System (page replacement)
- Keep pointer to last examined page frame
- Traverse pages in circular buffer
- Clear use bits as we search 
- Stop when find page with already cleared use bit, replace this page 
VAX/VMS second-chance lists	Problem addressed: 
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
FIFO (First-in-first-out) 	Replace page that has been in memory the longest. 

– Intuition: First referenced long time ago, done with it now
– Advantages: Fair: All pages receive equal residency; Easy to implement
– Disadvantage: Some pages may always be needed