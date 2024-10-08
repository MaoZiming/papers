# The Design and Implementation of a Log-Structured File System (1991) 

Link: https://people.eecs.berkeley.edu/~brewer/cs262/LFS.pdf

Read: July 11th, 2024

* **Sprite LFS** is a copy-on-write (COW) based file system that introduces **a new approach of writing** to disk: instead of overwriting file in place, **LFS buffers all updates into in-memory segment and write them out together sequentially**. LFS performs garbage collection periodically to reclaim free segments.  

* Compared to FFS: disk bandwidth is getting better. Computers are having more memory. Disk seek is still **really slow**. We can use some of the memory as buffer cache (help with the reads but not the writes).

* In this paper we present a solution based on large extents called segments, where a segment cleaner process continually **regenerates empty segments** by compressing the live data from **heavily fragmented segments**. 

## Motivation

* **System memories are growing**
    * More data can be cached in memory
    * More writes, FS performance is largely determined by write performance
* **Large gap between random I/O and sequential I/O performance**
  * Sequential I/O is decreasing much slower. 
* **Existing FS perform poorly on many common workloads**
    * E.x. FFS perform many writes to create a new file of size one block
* **File systems are not RAID-aware**
    * RAID-4 and RAID-5 have small-write problem (i.e. one logical write causes 4 physical I/O)

Thus we want to build an FS that focus on write performance, and try to make use of the sequential bandwidth of the disk.


## Motivation: optimize writes 
* System memories are growing, can be used for cache
* Existing FS perform poorly on common workloads: small-write problem 
    *  **FFS performs many writes to create a small file (5 writes)**
    *  Too many small writes. 
* When writing small files in such a system, less than 5% of the disk’s potential bandwidth is used for new data; the rest of the time is spent seeking.
   * Another problem is synchronous writes. 
* Unix FFS writes file data blocks asynchronously, file system metadata structures such as directories and inodes are written synchronously

* a log-structured file system converts the **many small synchronous random writes** of traditional file systems into **large asynchronous sequential transfers** that can utilize nearly 100% of the raw disk bandwidth.

The crux is: how can a file system **tranform all writes into sequential writes**? how to read data, and how to free space? 

## How it works 

### Sprite LFS outputs index structures in the log to permit random-access retrievals.

### Inode map

Problem: finding inode is hard in LFS 

* FFS finding **inode**: easy, organized, put at fixed location
  * This is really nice!
* LFS: scatter the inodes all throughout the disk! Worse, we never overwrite in place, and thus the latest version of an inode (i.e., the one we want) keeps moving.

Solution: **inode map**

* **Inode map**: takes an **inode number** as input and produce disk address of most recent address of the inode
* Inode map: **persistent**
    * Keep track of locations of inodes across crashes
    * Chunks of inode map is placed right next to where it is writing all of the other new information
* Inode map tracks inode locations across segments. 

Unfortunately, as it gets updated frequently, this would then require updates to file structures to be followed by writes to the imap, and hence performance would suffer (i.e., there would be more disk seeks, between each update and the fixed location of the imap).


### 1.5 Checkpoint Region

* FS have fixed and known location on disk to begin file lookup: **checkpoint region (CR)**
* Contains **pointers to the latest pieces of the inode map**
* Updated periodically (~30s) or so

* LFS should perform the same amount of I/O as a typical file system during read
* The entire **imap** is cached so extra work is just look up the inode’s address in the imap
* Thus, the overall structure of the on-disk layout contains a checkpoint region (which points to the latest pieces of the inode map); the inode map pieces each contain addresses of the inodes; the inodes point to files (and directories) just like typical UNIX file systems.
* Inode map contains the information for the location of both the directory file *dir* as well as the newly created file *f*

LFS uses write buffering to keep track of updates in memory before writing to disk 
  *  _Segment_: unit of buffering
  *  How to read?
      *  _Inode map_: **inode number maps to the disk address of the inode**
          *  Placed right next to where it is writing all other new information
      *  _Checkpoint region_: contains pointers to the latest pieces of inode map
          *  Updated periodically
          *  Read the entire inode map, and cache in memory
          *  Read: same I/Os as the typical file system
      *  Steps to read
          *  Read **CR**
          *  Read **inode map and cache the inode map in memory.**
          *  Given inode #
              *  Look up inode map, read inode
              *  Read block
       *  ![checkpoint-region](images/39-lfs/checkpoint-region.png)
* ![alt text](images/39-lfs/lfs_vs_ffs.png)
  
## Garbage collection

* Threading vs. Copying
  * Threading: leave the live data in place and *thread* the log through the free extents.
    * The log skips over the active blocks and overwrites blocks of files that have been deleted or overwritten. Pointers between the blocks of the log are maintained so that the log can be followed during crash recovery. 
    * Threading leads to severe segmentation. 
  * Copying: copy live data out of the log in order to leave large free extents for writing.

*  Segment-level cleaning
   *  LFS uses both threading and copying in larger units of segments. 
   *  The log is threaded on a segment-by-segment basis; 
    *  LFS read in # of partially used segments, determine which blocks are live
    *  Compact: write out new set of segments with just live blocks
*  Policies
    *  Clean cold segments sooner and hot segments later   
    *  Free up old ones for writing
*  Determine block liveness with _segment summary block_
    *  Segment summary block is part of each segment. 
    *  For each file data block the **summary block** contains the file number and block number for the block.
   *  Cleaner is **very** workload dependent. 
      *  LFS forsakes read locality. 
*  The version number combined with the inode number form an unique identifier (uid) for the contents of the file. The segment summary block records this uid for each block in the segment; if the uid of a block does not match the uid currently stored in the inode map when the segment is cleaned, the block can be discarded immediately without examining the file’s inode.
      
## Crash consistency 
*  LFS organizes writes in a **log**
    *  i.e. CR points to head and tail segment, each segment points to next segment to be written
    *  A "log" is a linear sequence of segment on disk where each segment is a chunk of data and metadata written together in an append-only fashion
    *  LFS periodically update checkpoint region
    *  Crashes can happen in between 
*  Write to checkpoint regions fails
    *  Keeps two CR, write to them alternatively
    *  Only overwrite on checkpoint at a time 
    *  Use checksum / **timestamps** to identify the newest checkpoint
          * it first writes out a header (with **timestamp**)
          * then the body of the CR
          * then finally one last block (also with a **timestamp**)
          * If the system crashes during a CR update, LFS can detect this by seeing an inconsistent pair of **timestamps**  
    *  Detect crash with inconsistent timestamps 
*  Write to segment fails: roll-forward  
    *  Start with the last CR, find the end of log
    *  Use that to read through the next segments and see any valid updates
    *  If so, update FS  

# Log File System (LFS)

* LFS introduces a new approach of writing to disk
  * Instead of overwriting files in place, write to an unused portion of the disk, later reclaims that old space through cleaning
  * In DB this is called: **shadow paging**, and in FS it is called **copy-on-write**
    * Copy-on-wrtie: duplicate data only when changes are made
    * The old data isn't immediately "erased" but is instead "shadowed" by the new data until the space is reclaimed
    * Enables highly efficient writing
    * Gather all updates into an in-memory segment and write them out together sequentially
* No free-block list or bitmap in Sprite.

### Segments

* We present a solution based on large extents called segments, where a segment cleaner process continually regenerates empty segments by compressing the live data from heavily fragmented segments. 

### 1.6 Directories

* LFS also solve a problem: recursive update
    * Problem arises to any FS that never updates in place, but rather move updates to new locations on the disk
    * When an inode is updated, its location on disk changes (need to update way up to the file system tree)
    * With inode map, even though the location of inode might change, the change is never reflected on the directory itself
    * The imap is updated while the directory hold the same name to inode number mappings

## Issues with updates

### Solution 1: versioning 

* Keep older versions and allow users to restore old file system

### Solution 2: garbage collection 

* Imagine we append D1 to the original file k.
* In this case, a new version of the inode is generated, but the old data block is still pointed to by the inode. 
  * ![alt text](images/39-lfs/inode-garbage.png)
* LFS instead keeps only the latest live version of a file
* Cleaner works on a segment-by-segment basis
    * How it works
        * Periodically, LFS cleaner reads in a number of old (partially-used) segments
        * Determines which blocks are live within these segments
        * Write out a new set of segments with just the live blocks within them
        * Freeing up old ones for writing
    * I.e. reading in M existing segments, **compact** their contents into N new segments (N < M)
* Mechanism: how to determine block liveness?
    * Maintain **segment summary block**
    * Information, **for each data block D, its inode number (which file it belongs to) and its offset (which block of the file)**
      * This information is recorded in a structure at the head of the segment known as the segment summary block.
    * How it works
        * For block D located at disk address A
        * Look into segment summary block and find its inode number N and offset T
        * Look in imap to find where N lives and read N from disk
        * Using offset T, look in the inode, see where the inode thinks it is on the disk
        * If the pointer is at address A, then live; otherwise, dead
* Policy: how often should the cleaner run, and which segments to pick to clean?
    * Original LFS paper: segregate hot and cold segments
    * Hot: contents being frequently updated
    * Cold: few dead blocks, but rest of the contents are stable
    * Policy: clean cold segments sooner and hot segments later
        * Write cost = average amount of time the disk is busy per byte of new data written, including all the cleaning overheads
        * Write cost = total amount of bytes moved to and from disk divided by # of those bytes represent new data

### 1.8 Crash Recoveries

* What happens if the system crashes while LFS is writing to disk?
* LFS organizes these writes in a **log**
    * CR points to a head and tail segment, each segment points to the next segment to be written
    * Periodically update the CR
* What happens
    * **Writing to the CR fails**
        * LFS keeps two CRs, one at either end of the disk, and write to them alternately
        * Implements a protocol when updating the CR with the latest pointers to the inode map and other information
            * First writes out a header (Timestamp)
            * Then the body of the CR
            * Then finally one last block (Timestamp)
            * If crashes, see inconsistent timestamps
 - **Writing to a segment fails**
    * Write CR every 30s or so, so last consistent snapshot of FS may be quite old
    * Upon reboot, recover by reading in CR, but it is possible many seconds of updates would lost
    * Solution: **roll forward**
        * Start with the last checkpoint region
        * Find the end of the log (which is included in the CR)
        * Use that to read through the next segments and see if there are any valid updates within it
        * If so, updates the FS accordingly and recovers much of the data and metadata written in the last checkpoint

### Performance

* Excellent performance on many different devices
    * Hard drives: position time is minimized
    * Parity-based RAIDs (-4, and -5), avoid small write problem entirely
    * Flash-based SSDs
* Downside: generate garbage
    * **Cleaning** is hard, cleaning cost is a concern
* Some modern commercial FS: NetApp’s WAFL, Sun’s ZFS, Linux btrfs, flash-based SSDs
    * Adopt similar copy-on-write approach
    * WAFL gets around cleaning problem by turning them into feature: snapshots!
