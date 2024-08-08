# Application-Controlled Physical Memory using External Page-Cache Management (1992)  
This paper proposes a virtual memory system that provides application control of physical memory using external page-cache management. 

Link: https://dl.acm.org/doi/pdf/10.1145/143365.143511

Read: June 30th, 2024.

## Key Insights

* Application control of physical memory. In this approach, a sophisticated application is able to **monitor and control the amount of physical memory it has available for execution**, the exact contents of this memory, and the scheduling and nature of page-in and page-out under the abstraction of a page frame cache provided by the kernel. 

* For example, a database management system can ensure that critical pages, **such as those containing central indices and directories, are in physical memory**. The query optimizer and transaction scheduler can also benefit from knowing which pages are in memory, because the cost of a page fault can increase the overall cost of a query.

### Main Problem: want application control 

* Application cannot know the amount of physical memory it has available 
    * Application is not informed when changes are made 
    * It cannot control physical pages it is allocated 
* A program cannot efficiently control the contents of physical memory allocated to it 
* A program cannot easily control the **read-ahead**, **writeback**, and discarding of pages within its physical memory 
  * A default process-level manager provides page-cache management for applications that do not want to manage their virtual memory. 

## Main Idea
* Virtual memory system provides application with one or more physical page caches that the application **can manage external to the kernel**. In particular, **it can know the exact size of the cache in page frames**, control which page is selected for replacement, and how data is transferred in and out of the cache. It also has information about physical addresses to implement schemes like page coloring and physical placement control. 

## Main techniques
* Kernel Modifications: Simple extensions to the kernel to allow user-level code to manage the page cache.

* APIs for Memory Control: A set of application programming interfaces (APIs) that **let the application instruct the kernel** on how to manage the page cache.

* User-level Page Fault Handling: The paper argues that the cost of a page fault is too high to be abstracted away by the kernel, so user-level fault handling might be incorporated to decrease this time.

* Dynamic Algorithms: The application might use special algorithms to decide when to evict or keep certain pages in the cache.

## V.s Microkernel 
* Similarity in simplified kernel by allowing applications to manage their own physical memory cache external to the kernel, and giving higher-degree of control to the user space. 
