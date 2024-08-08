# Lightweight Recoverable Virtual Memory (1993)

Link: https://people.eecs.berkeley.edu/~brewer/cs262/lrvm.pdf

Read: June 25th, 2024.

This paper proposes an efficient, flexible, and ease-to-use implementation of **recoverable virtual memory (RVM)** to manage persistence for critical data structure. A UNIX programmer thinks of LRVM as just a typical subroutine library. 

* Recoverable virtual memory refers to regions of a virtual address space on which transactional guarantees are offered.

* Goal: allow Unix applications to manipulate persistent data structures (such as the meta data for a file system) in a manner that has clear-cut failure semantics.

## Prior works
* Existing solutions---such as Camelot---were too heavy-weight. Wanted a "lite" version of these facilities that didn't also provide (unneeded) support for distributed and nested transactions, shared logs, etc.
* Solution: a **library package** that provides only recoverable virtual memory

Lessons from Camelot:
* Overhead of multiple address spaces and constant **IPC** between them was significant. Heavyweight facilities impose additional onerous programming constraints.
* **Size and complexity of Camelot** and its dependence on special Mach features resulted in maintenance headaches and lack of portability. (The former of these two shouldn't be an issue in a ``production'' system.)
* Camelot had a yucky object and process model. Its componentization led to lots of IPC. It had poorly tuned log truncation. Was perhaps too much of an embrace of Mach.
* However, note that the golden age of CMU Systems learned a lot from the sharing of artifacts: Mach, AFS, Coda...
* A lot of positive spirit in this paper.

## Lightweight TxN: simplicity over generality 
* Eliminate support for nesting and distribution
* Factor out concurrency control, let applications handle it
* Can kernel or user threads
* Factor out resiliency to media failure (i.e. mirror)

## Interface 
* initialize, map, unmap
* begin_txn, end_txn, set_range, abort_txn
    * abort: need to undo the change in memory (with no `no-store`)
    * end_txn: if `no-flush`, then lazy commit
* flush, truncate
* query_options, set_options, create_log
* `no-store`: contents of old-value records do not have to be copied or buffered
* `no-flush`: new-value and commit records and be spooled rather than forced to log
  
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

* **External Data Segment**: Permanent storage.
* Data moved here when the **transaction commits** or during optimization processes like truncation.

### **Crash Recovery**

* Read the disk-based Redo Log in reverse.
* Apply changes to the External Data Segment.
* Truncate the log to free up space (only wait until all changes updated in External Data Segment)
