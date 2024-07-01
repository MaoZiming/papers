# The Design and Implementation of a Log-Structured File System (1991) 
LFS is a copy-on-write (COW) based file system that introduces **a new approach of writing** to disk: instead of overwriting file in place, LFS buffers all updates into in-memory segment and write them out together sequentially. LFS performs garbage collection periodically to reclaim free segments.  

## Motivation: optimize writes 
1. System memories are growing, can be used for cache
2. Existing FS perform poorly on common workloads: small-write problem 
    *  FFS performs many writes to create a small file (5 writes)

The crux is: how can a file system tranform all writes into sequential writes? how to read data, and how to free space? 

## How it works 
LFS uses write buffering to keep track of updates in memory before writing to disk 
  *  _Segment_: unit of buffering
  *  How much to buffer?
      *  write pays position cost, how much to write in order to amortize the cost
  *  How to read?
      *  _Inode map_: inode number maps to the disk address of the inode
          *  Placed right next to where it is writing all other new information
      *  _Checkpoint region_: contains pointers to the latest pieces of inode map
          *  Updated periodically
          *  Read the entire inode map, and cache in memory
          *  Read: same I/Os as the typical file system
      *  Steps to read
          *  Read CR
          *  Read inode map and cache
          *  Given inode #
              *  Look up imap, read inode
              *  Read block
<img width="617" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/51ea5ecd-08e4-4a14-9b32-208821ea95b6">

     
## Garbage collection 
*  Segment-level cleaning
    *  LFS read in # of partially used segments, determine which blocks are live
    *  Compact: write out new set of segments with just live blocks
*  Policies
    *  Clean cold segments sooner and hot segments later   
    *  Free up old ones for writing
*  Determine block liveness with _segment summary block_
    *  SS[Data Block] = (inode #, offset)
      
## Crash consistency 
*  LFS organizes writes in a **log**
    *  i.e. CR points to head and tail segment, each segment points to next segment to be written
    *  A "log" is a linear sequence of segment on disk where each segment is a chunk of data and metadata written together in an append-only fashion
    *  LFS periodically update checkpoint region
    *  Crashes can happen in between 
*  Write to checkpoint regions fails
    *  Keeps two CR, write to them alternatively
    *  Only overwrite on checkpoint at a time 
    *  Use checksum / timestamps to identify the newest checkpoint
          * it first writes out a header (with timestamp)
          * then the body of the CR
          * then finally one last block (also with a timestamp)
          * If the system crashes during a CR update, LFS can detect this by seeing an inconsistent pair of timestamps  
    *  Detect crash with inconsistent ts 
*  Write to segment fails: roll-forward  
    *  Start with the last CR, find the end of log
    *  Use that to read through the next segments and see any valid updates
    *  If so, update FS  
