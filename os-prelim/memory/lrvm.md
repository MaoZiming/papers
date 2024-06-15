# Lightweight Recoverable Virtual Memory (1993)
This paper proposes an efficient, flexible, and ease-to-use implementation of recoverable virtual memory (RVM) to manage persistence for critical data structure. A UNIX programmer thinks of LRVM as just a typical subroutine library. 

## Lightweight TxN: simplicity over generality 
- Eliminate support for nesting and distribution
- Factor out concurrency control, let applications handle it
- Can kernel or user threads
- Factor out resiliency to media failure (i.e. mirror)

## Interface 
- initialize, map, unmap
- begin_txn, end_txn, set_range, abort_txn
    - abort: need to undo the change in memory (with no `no-store`)
    - end_txn: if `no-flush`, then lazy commit
- flush, truncate
- query_options, set_options, create_log
- `no-store`: contents of old-value records do not have to be copied or buffered
- `no-flush`: new-value and commit records and be spooled rather than forced to log
  
## Steps

### **Program Flow**

1. **Start (begin_txn)**:
    - **Undo Log** created in-memory to store original data.
    - **Redo Log** created in-memory to store new changes.
2. **During Transaction**:
    - Undo Log holds original data.
    - Redo Log records changes made.
3. **End Transaction (end_txn or abort_txn)**:
    - **Commit (end_txn)**: Redo Log moves from memory to disk for durability, then clears in-memory logs.
    - **Abort (abort_txn)**: Undo Log restores original data, discards both logs.

### **Data Storage**

- **External Data Segment**: Permanent storage.
- Data moved here when the transaction commits or during optimization processes like truncation.

### **Crash Recovery**

- Read the disk-based Redo Log in reverse.
- Apply changes to the External Data Segment.
- Truncate the log to free up space (only wait until all changes updated in External Data Segment)
