# Application-Controlled Physical Memory using External Page-Cache Management (1992)  
This paper proposes a virtual memory system that provides application control of physical memory using external page-cache management. 

## Key Insights

### Main Problem: want application control 

1. Application cannot know the amount of physical memory it has available 
    1. It is not informed when changes are made 
    2. It cannot control physical pages it is allocated 
2. A program cannot efficiently control the contents of physical memory allocated to it 
3. A program cannot easily control the read-ahead, writeback, and discarding of pages within its physical memory  

## Main Idea
Virtual memory system provides application with one or more physical page caches that the application can manage external to the kernel. In particular, it can know the exact size of the cache in page frames, control which page is selected for replacement, and how data is transferred in and out of the cache. It also has information about physical addresses to implement schemes like page coloring and physical placement control. 

## Main techniques
Kernel Modifications: Simple extensions to the kernel to allow user-level code to manage the page cache.

APIs for Memory Control: A set of application programming interfaces (APIs) that let the application instruct the kernel on how to manage the page cache.

User-level Page Fault Handling: The paper argues that the cost of a page fault is too high to be abstracted away by the kernel, so user-level fault handling might be incorporated to decrease this time.

Dynamic Algorithms: The application might use special algorithms to decide when to evict or keep certain pages in the cache.

## V.s Microkernel 
Similarity in simplified kernel by allowing applications to manage their own physical memory cache external to the kernel, and giving higher-degree of control to the user space. 
