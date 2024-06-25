# JFS: Crash Consistency 
**_setup_**: FS with inode bitmap, data bitmap, inodes, and data blocks 

**_scenario_**: append requires updating inode, the data block, and data bitmap. Imagine the dirty writes still sits in main memory in the page cache for some times. 
*  If just data is written 
    *  No inode points to it, no bitmap says block is allocated
    *  No problems with FS crash consistency!
*  If just updated inode or bitmap is written
    *  FS inconsistency: bitmap and inode

## Crash Consistency Problem 
We want to move the FS from one consistent state to another **atomically**. 

Unfortunately, we can’t do this easily because the disk only commits one write at a time, and crashes or power loss may occur between these updates. We call this problem **crash-consistency problem** (i.e. consistent update problem). 

## Solution 1: FSCK
*  A UNIX tool to check and repair a disk partition
*  Basic idea: let inconsistency happen, fix them later
*  Problem: complicated, slow, inefficient (i.e. search-entire-house-for-keys recovery) 

## Solution 2: Journaling (write-ahead logging) 
_example_: Linux ext3, ext4, IBM’s JFS, CFS, Windows NTFS

_basic_idea_: when updating disk, before overwriting structures in place, first write down a log describing what you are able to do.
### Redo-logging
The key idea is to go back and try again with this log. 

### Data journaling 
*   Steps
    *   Journal write: write TxB, metadata, and data to log, wait for complete
    *   Journal commit: write TxE, wait for complete
    *   Checkpoint: write contents of updates (metadata, data) to final on-disk locations
*   Problem: slow, most data written twice 

### Metadata (ordered) journaling 
*    E.x. NTFS, XFS, more popular
*    Problem: concurrencly write data and metadata independently, file system may point to garbage data
*    Idea: write data blocks to disk first
     *  Data write
     *  Journal metadata write
     *  Wait for step (1) and (2) complete
     *  Journal commit
     *  Checkpoint metadata        
### Performance v.s fault tolerance tradeoffs
Journaling techniques add some overheads to each write but recovers more quickly from power loss. 

## Other Solutions 
1. Software updates
2. **Copy-on-write (COW)**: **like in LFS**
   *   Technique: never overwrite files or directories in place
   *   Place new updates to unused locations
   *   After a # of updates, COW FS filps root structure of FS to include pointers
3. Optimistic crash consistency
   *   Issue as many writes to disk as possible by using transaction sum  
