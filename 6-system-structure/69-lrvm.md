# Lightweight Recoverable Virtual Memory (1993)

Link: https://people.eecs.berkeley.edu/~brewer/cs262/lrvm.pdf

Read: June 25th, 2024.

This paper proposes an efficient, flexible, and ease-to-use implementation of **recoverable virtual memory (RVM)** to manage persistence for critical data structure. A UNIX programmer thinks of LRVM as just a typical subroutine library. 

* Recoverable virtual memory refers to regions of a virtual address space on which transactional guarantees are offered.

* Goal: allow Unix applications to manipulate **persistent** data structures (such as the meta data for a file system) in a manner that has clear-cut failure semantics.

* This transactional facility, called RVM, is implemented without specialized operating system support, and has been in use for over two years on a wide range of hardware from laptops to servers.
* RVM is intended for Unix applications with persistent data structures that must be updated in a fault-tolerant manner. The total size of those data structures should be a **small fraction of disk capacity**, and their working set size must easily fit within main memory.

> Our design challenge lay not in conjuring up features to add, but in determining what could be omitted without crippling RVM.

* We placed data structures pertaining to Coda meta-data in recoverable memory on servers. The meta-data included Coda directories as well as persistent data for replica control and internal housekeeping. The contents of each Coda file was kept in a Unix file on a server’s local file system. Server recovery consisted of Camelot restoring recoverable memory to the last committed state, followed by a Coda salvager which ensured mutual consistency between meta-data and data.

* **Backing Store**: RVM’s backing store for a recoverable region, called its external data segment, is completely independent of the region’s VM swap space.

* Simplicity over generality. 

## Log for each process

* each process using RVM has a separate log. The log can be placed in a Unix file or on a raw disk partition. When the log is on a file, RVM uses the fsync system call to synchronously flush modifications onto disk. RVM’s permanence guarantees rely on the correct implementation of this system call. For best performance, the log should either be in a raw partition on a dedicated disk or in a file on a log-structured Unix file system.

## Prior works
* Existing solutions---such as Camelot---were too heavy-weight. Wanted a "lite" version of these facilities that didn't also provide (unneeded) support for distributed and nested transactions, shared logs, etc.
* Solution: a **library package** that provides only recoverable virtual memory

Lessons from Camelot:
* Overhead of multiple address spaces and constant **IPC** between them was significant. Heavyweight facilities impose additional onerous programming constraints.
* **Size and complexity of Camelot** and its dependence on special Mach features resulted in maintenance headaches and lack of portability. (The former of these two shouldn't be an issue in a ``production'' system.)
* Camelot had a yucky object and process model. Its componentization led to lots of IPC. It had poorly tuned log truncation. Was perhaps too much of an embrace of Mach.
* However, note that the golden age of CMU Systems learned a lot from the sharing of artifacts: Mach, AFS, Coda...
* A lot of positive spirit in this paper.
* Increase a third of CPU cycles on the server. 

## Segments and regions

* Recoverble memory is managed in *segments*. 
* The backing store for a segment is called its *external data segment*.
  * It may be either a file or a raw disk partition. 
* The copying of data from external data segment to virtual memory occurs when a region is mapped. 
* Regions can be unmapped at any time, as long as they have no uncommitted transaction outstanding. 

## Linked structure

* we have structured RVM as a library that is linked in with an application. No external communication of any kind is involved in the servicing of RVM calls.

## Structure
* IPC is 600x slower than local procedure calls
* Thus, LRVM will be **linked** with your program code
* In a log-structured UNIX file system - placing one RVM log per app would be very performant

## Lightweight TxN: simplicity over generality 
* Eliminate support for nesting and distribution
* Factor out concurrency control, let applications handle it
* Can kernel or user threads
* Factor out resiliency to media failure (i.e. mirror)

## Interface 
* initialize, map, unmap
* begin_txn, end_txn, set_range, abort_txn
    * abort: need to undo the change in memory (with no `no-store`)
    * `set_range` : lets RVM knows that a certain area of a region is about to be modified. 
      * Allows RVM to record the current value of the area so that it can undo changes in case of an abort. 
    * end_txn: if `no-flush`, then lazy commit
    * abort_txn: abort transaction. 
* flush, truncate
* query_options, set_options, create_log
* `no-store`: contents of old-value records do not have to be copied or buffered
* `no-flush`: new-value and commit records and be spooled rather than forced to log
* Bounded persistence: bound the amount of time to flush the transaction to disk. 

## Steps

### **Program Flow**

* **Start (begin_txn)**:
    * **Undo Log** created in-memory to store original data.
    * **Redo Log** created in-memory to store new changes.
* **During Transaction**:
    * Undo Log holds original data.
    * Redo Log records changes made.
* **End Transaction (end_txn or abort_txn)**:
    * **Commit (end_txn)**: Redo Log moves from memory to disk for durability, then clears in-memory logs.
    * **Abort (abort_txn)**: Undo Log restores original data, discards both logs.

### **Data Storage**

* **External Data Segment**: **Permanent storage**.
* Data moved here when the **transaction commits** or during optimization processes like truncation.

### **Crash Recovery**

* Read the disk-based Redo Log in reverse.
* Apply changes to the External Data Segment.
* Truncate the log to free up space (only wait until all changes updated in External Data Segment)


* Crash recovery consists of RVM first reading the log from tail to head, then constructing an in-memory tree of the latest committed changes for each data segment encountered in the log. The trees are then traversed, applying modifications in them to the corresponding external data segment. Finally, the head and tail location information in the log status block is updated to reflect an empty log.
