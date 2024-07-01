# Storage Systems
- Physical organization and performance of Hard Disk Drives (HDDs) and Solid State Drives (SSDs)
- File system structure: directory structure, index structure (inodes), storage blocks, free space map, hard links, soft links, and buffer cache
- Case studies: File Allocation Table (FAT) and Berkeley Fast File System (FFS)
- Replication as a means for durability: erasure codes and RAID (disk replication)
- Transactions as a means for reliability: journaling file systems and redo logging

## Concepts explain

### HDDs

- Disk has a sector-addressable address space
    - Array of sectors, each typically 512 bytes (i.e. blocks)
    - Main operations: reads and writes to sectors
- Basic geometry
    - Platter: circular hard surface on which data is stored persistently by including magnetic change to it
        - 2 sides: each side is a surface
    - Spindle: platters bound together around spindle (i.e. rotations)
    - Track: data encoded on each surface in concentric circles of sectors, one circle is a track
    - Disk head (perform read and write) attached to a single disk arm
- Performance
    - Read / write time = seek time (multiple track) + rotation (single track latency) + transfer time
    - Mechanical and slow

### SSDs

- Basics
    - A **flash chip** consists of many banks, each of which is organized into **blocks**
    - Each blocks is further subdivided into some number of **pages**
    - Blocks are large (128KB - 2MB) and contain many pages of small size (1-8KB)
- **Organizations**
    - Some number of flash chips (for persistent storage)
        - Use multiple flash chips in parallel, reduce write amplifications
    - Some amount of volatile (i.e. non-persistent) memory (e.g. SRAM)  for caching
    - Control logic to orchestrate device operation: FTL
        - Transform reads and writes from a client into reads, erases, and programs to underlying flash chips
- R/W
    - Read: issue read with an address and length
    - Write: client erase the entire block (which delete information within the block), then client can program each page exactly once
- **Performance**
    - Read
        - ~10ms, regardless of location of device, super good read performance (i.e. random access device)
    - Write
        - Erase is expensive
- **Reliability**
    - Primary concern: wear out
        - Explain: if a block is erased and programmed too often, it becomes unusable
        - Sol: *wear leveling*
            - FTL should try its best to spread that work across all the blocks of the device evenly
            - Sol: Log-structured FTL
                - Spread out the write load (v.s. overwriting the same block)
                - Garbage collection helps as well
                    - I.e. finding a block that contains garbage pages, read in live pages, write out those to log, reclaim entire block in writing
    - Another issue: *disturbance*
        - When accessing a particular page, it is possible that some bits get flipped
        - Sol: program pages within an erased block in order, from low page to high page

### SSD v.s. HDD comparisons

- Performance
    - Much better in **random reads and random writes** than HDD
    - **Sequential performance is less of a different**
    - SSD random read not as good as random write
        - Why unexpectedly good random write performance?
            - Due to log-structured design of many SSDs, which transform random writes into sequential ones to improve performance
    - **Still gaps in sequential and random access performance**
- Cost: per unit of capacity
    - Prevent usage of SSD
    - SSD: $150 for a 250GB drive, 60 cents per GB
    - HDD: $50 for 1TB, 5 cents per GB


## File System Structure

- **File:** an array of bytes which can be created, read, written, and deleted
    - It has a low-level name (i.e. a number) that refers to it uniquely
    - The low-level name is often called an i-number
- **Directory:** a collection of tuples, each of which contains a human-readable name and low-level name in which it maps
    - Each entry refers to either another directory or to a file
    - Each directory has a low-level name (i-number) itself
    - Always have two special entries: `.` and `..`
- **Inode:** index node
    - Structure that holds the metadata for a given file (i.e. length, permissions, locations of its blocks)
- **Storage blocks**
    - Data structure of the file system to organize data and metadata
    - Arrays of blocks
    - Block: commonly 4KB
    - FS organization
        - Data region: region of disk used for user data
        - Metadata: keep track of information about each file
        - Allocation structure: whether inodes or data blocks are freed or allocated
        - Superblock: contain information about a specific fs
- **Free space management**
    - The system must track which inodes and data blocks are free s.t. when new file / directory is created, it is able to find space for it
    - Two bit maps
        - Inode bitmap: find an inode that is free, allocate it to a file, marked it as used
        - Data bitmap: find data block that is free, allocate it
- **Links**
    - **Hard link:** create another name in the directory you are creating the link to, and refer it to the same inode number (i.e. low-level name) of the original file
        - Data is not copied, just two names
        - Use reference counting: inode maintains counter that allow FS to check how many different file names have been linked to the nodes
        - Pros
            - Can’t create one to directory (i.e. fear of cycle in directory tree)
            - Can’t hard link to files in other disk partitions (i.e. inode number only unique within a single FS)
    - **Symbolic link:** actually is a file itself that point to another file by name
        - I.e. holding the pathname of the linked-to file as the data of the link file
        - Removing the original file causes the link to point to a pathname that no longer exists: dangling pointers
- **Buffer cache**
    - Or page cache
    - Refer to an area of main memory (RAM) that temporarily holds copies of disk blocks (data blocks, inodes, imaps, etc.)
    - The use of a cache aims to reduce the # of slow disk I/O operations by keeping frequently accessed or recently used data in faster, but more limited, main memory


## Case Study

### File Allocation Table (FAT)

- **How to design an inodes to point to data blocks?**
- Baseline: use linked list
    - One pointer, points to the first block of the file, and add another pointer at the end of that data block, etc. to support large files
    - Poor performance: random access, or access offset of the file, etc.
- File Allocation Table
    - In-memory table of link information instead of storing next pointers with the data blocks
    - Indexed by address of data block $D$
    - Content of entry: $D$’s next pointer (i.e. the address of next block in file which follows $D$) d
        - Marker to indicate EOF and whether a particular block is free
    - Directory entries
        - No inode per se
        - Directory entries that store metadata about a file and refer directly to the first block of said file
- How does this compare with inode-based structure?
    - In memory mapping then eliminate the need of traversal
    - But not in-memory, then inode-structure has much better random read performance
        - And also inode-structure can have hard links

### Fast File System (FFS)

First disk-aware file systems: 

- Motivation
    - Old UNIX file system structure has bad performance (i.e. data spread all over the place, data blocks away from inode)
    - File system get fragmented, free list ended up pointing to bunch of blocks spread across the disk
    - Original block size too small
- The idea was to design the file system structures and allocation policies to be “disk aware” and thus improve performance
- Organizing structure: cylinder group (i.e. locality groups)
    - Use group across disks
        - Groups are ranges of cylinders
    - Keep related stuff together: allocate inodes and data blocks in the same group
        - File inodes are allocate in the same group with directory
        - Directory inodes are allocated in new group with fewer used inodes than average group
        - First data blocks allocate near inodes
        - Other data blocks allocated near previous blocks
- Usability
    - Long file names
    - Atomic rename
    - Symbolic links

## Replication for Durability: Erasure Codes & RAID

### Erasure Code

- A method used in storage system to improve durability and availability of data
- Erasure coding is a process where a file is broken into multiple fragments, and extra redundant fragments are generated from the original fragments
    - These fragments are then stored across different locations (e.g., disks, servers, etc.)
    - The magic of erasure coding is that you can reconstruct the original file from a subset of its fragments. This means that even if some fragments are lost or corrupted, you can recover the complete file.

### RAID: redundant array of inexpensive disk

- RAID: a faster, larger, and more reliable disk system
    - One logical disk built from many physical disk
    - Workloads: R/W, fail-stop fault model
- RAID-0: striping
    - Optimize for capacity, no redundancy
- RAID-1: mirroring
    - Keep mirrored copies of the data
- RAID-4: use a parity disk
    - Parity: a form of redundancy used to provide fault tolerance; calculated from the actual data block, purpose is to allow reconstruction of missing or corrupted data
    - Small write problem: parity disk is the bottleneck
- RAID-5: rotating parity
    - Random write improves!

<img width="741" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/33a919cc-95f3-4a90-a70e-b4109bc1d754">

## Transactions as a means for reliability: journaling file systems and redo logging

### Journal File Systems

- **Key problem:** how to update the disk despite crashes?
    - The system may crash or lose power between any two writes, the on-disk state may get partially updated
    - Example
        - Append operation
            - Open the file
            - `lseek()` to move the file offset to end of the file
            - Issue a 4KB write to the file before closing it
            - This needs to update
                - The inode (which must point to new block and record the new larger size due to append)
                - The new data block
                - A new version of the data bitmap
    - One single write succeeds
        1. Just the **data block (Db)** is written to disk
            1. Just data is on disk, no inode points to it, no bitmap says the block is allocated 
            2. As if the write never occurs, no problem at all from FS crash consistency point! 
        2. Just the **updated inode** is written to disk
            1. We will read garbage data from the disk! 
            2. New problem: **FS inconsistency** 
                1. On-disk bitmap is telling us that data block has not been allocated, but inode saying that it has 
        3. Just the **updated bitmap** is written to disk 
            1. File system is inconsistent again 
            2. Bitmap indicate that the block is allocated, but no inode points to it 
            3. This write will result in **space leak**, as block 5 would never be used by FS
    - After the crash, the system boots and wishes to mount the FS again
    - Given that crashes can occur at arbitrary points in time, how do we ensure the file system keeps the on-disk image in a reasonable state?
- **Crash consistency** problem
    - What we’d like to do ideally is move the file system from one consistent state (e.g., before the file got appended to) to another **atomically** (e.g., after the inode, bitmap, and new data block have been written to disk).
    - Unfortunately, we can’t do this easily because the disk only commits one write at a time, and crashes or power loss may occur between these updates. We call this problem **crash-consistency problem** (i.e. **consistent update problem**).
- Journal File System
    - Basic idea: when updating the disk, before overwriting structures in place, first write down a little note (i.e. log) describing what you are about to do
    - Steps (data journaling)
        1. **Journal write:** Write the contents of the transaction (including TxB, metadata, and data) to the log; wait for these writes to complete.
        2. **Journal commit:** Write the transaction commit block (containing TxE) to the log; wait for write to complete; transaction is said to be **committed**.
        3. **Checkpoint:** Write the contents of the update (metadata and data) to their final on-disk locations.
    - Steps (metadata journalling)
        1. **Data write:** Write data to final location; wait for completion (the wait is optional; see below for details).
        2. **Journal metadata write:** Write the begin block and metadata to the log; wait for writes to complete.
        3. **Journal commit:** Write the transaction commit block (containing TxE) to the log; wait for the write to complete; the transaction (including data) is now committed.
        4. **Checkpoint metadata:** Write the contents of the metadata update to their final locations within the file system.
        5. **Free:** Later, mark the transaction free in journal superblock.
    - The key about the above steps
        - Forcing data write to complete is not required for correctness
        - It will be fine to concurrently issue writes to data, the transaction begin log and journaled metadata
        - The real requirement: step 1 and 2 complete before issuing journal commit
            - rule of “**write the pointed-to object before the object that points to it**” is at core of crash consistency
    - Recovery
        - After commit, before checkpoint completes: recover update by replay the log (i.e. **redo logging**)
            - It completes transactions that were committed but not yet checkpointed and ignores or rolls back incomplete transactions.
