# File System Implementation: Very Simple File System 

## Overall Organization 

- **block**: commonly 4KB
- **How to store it?**
    - **Data region**: the region of disk used for user data
    - **Metadata:** keep track of information about each file
        - the size of the file, its owner and access rights, access and modify times, and other similar kinds of information
        - File systems usually have a structure called an inode
        - This portion of the disk is the **inode table**
    - **Allocation structure:** track whether inodes or data blocks are freed or allocated
        - E.x. free list: points to the first free block, then the next
        - E.x. **bitmap: data bitmap (d), inode bitmap (i)  (0 for free, 1 for in-use)**
        - Chosen in the example below
    - **Superblock:** contains the information about the particular file system
        - E.x. how many inodes and data blocks are in the file system, where the inode table begins, identify the file system type, etc.
    - ![alt text](image-7.png)
- E.x. when mounting a FS
    - OS will read the superblock first to initialize various parameters, then attach the volume to the file system tree
    - When files within the volume are accessed, the system will know exactly where to look for the needed on-disk structure

## File Organization: The Inode 

- Inode: index node 
  - Used because these nodes where originally arranged in an array, and the array indexed into when accessing a particular node
  - Generic name used in FS to describe the structure that holds the metadata for a given file (i.e. length, permissions, location of its blocks)
  - referred to with i-number. 
  - Each inode stores a bunch of pointers. 
- Information
    - Type: regular, directory, etc.
    - Size
    - Number of blocks allocated to it
    - Protection information (who owns the file, who can access it)
    - **Some time information (when the file was created, modified, last accessed)**
    - Where its data block reside on disk
        - How?
            - Direct pointers (disk addresses) inside each inode
              - We need an inode per data block.
              - For a 36KB file, if each data block is 4KB, we need 9 data blocks. and 9 direct pointers to these 9 data blocks. 
                - Cons: limited, you want to have file really big (i.e. bigger than block size)?
                - **Multi-level index: indirect pointer**
                    - Instead of pointing to a block that contains the user data
                    - Points to a block that contains more pointers, each of which point to the user data
                    - An inode may have some fixed number of direct pointers, and a single indirect pointer
                    - Even larger file: **double indirect pointer**
                        - Refers to a block that contains pointers to indirect blocks
                        - Each of which contain pointers to data blocks
                    - E.x. Linux ext2, ext3, UNIX file system
                - Others use **extents** instead of pointers
                    - Akin to segments in virtual memory
                    - Simply a disk pointer plus a length (in blocks)
                    - Instead of requiring a pointer for every block of a file, all one needs is a pointer and a length to specify the on-disk location of a file
                    - Just a single extent is limiting, as one may have trouble finding a contiguous chunk of on-disk free space when allocating a file. Thus, extent-based file systems often allow for more than one extent, thus giving more freedom to the file system during file allocation
                - **Pointer v.s Extent**
                    - Pointer: more flexible but use large amount of metadata per file (particularly for large file)
                    - Extent: less flexible but more compact, work well when there is enough free space on the disk and files can be laid out contiguously

## Why pointers at this time? i.e. imbalanced tree?
- Finding!
    - Most files are small, it makes sense to optimize for this case
        - Small # of direct pointers

### Others

- Linked list: instead of having multiple pointers, just need one to point to first block of the file
    - Performs poorly for some workloads: i.e. reading last block, or doing random access
- Some system keeps in-memory table of link information
    - Table indexed by address of data block D
    - Content of an array is D’s next pointer
    - Having such table makes it so that linked allocation scheme can effectively do random file accesses, by first scanning through (in-memory) table to find the desired block and accessing (on disk) it directly
    - Basics of **file allocation table (FAT) file system**
        - I.e. classic old Windows file system


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
        - 

## Directory Organization

- A directory basically just contains a list of (entry name, inode number) pairs
- For each file or directory in a given directory
    - There is a string and a number in the data block(s) of the directory
    - For each string, there may also be a length
- Each entry has an inode number, record length (the total bytes for the name plus any left over space), string length (the actual length of the name), and finally the name of the entry.

![alt text](image-8.png)


## Free space management

- The system must track which inodes and data blocks are free, and which are not
- When a new file or directory is created, it is able to find the space for it
- **Free space management** with two bit maps
    - First, search through bitmap for an inode that is free, and allocate it to the file, marked it as used, and update the bitmap eventually
    - Update the data block similarly
- **Pre-allocation policy**: look for contiguous blocks
  - For example, some Linux file systems, such as ext2 and ext3, will look for a sequence of blocks (say 8) that are free when a new file is created and needs data blocks; by finding such a sequence of free blocks, and then allocating them to the newly-created file, the file system guarantees that a portion of the file will be contiguous on the disk, thus improving performance.

## Access paths: reading and writing 

### Reading

- The file system must **traverse** the pathname and thus located the desired node
- All traversals begin at the root of the file system, in the **root directory**
- The amount of I/O generated by the open is proportional to the length of the pathname
    - For each additional directory in the path, read its inode and data

All traversals begin at the root of the file system, in the root directory which is simply called /. Thus, the first thing the FS will read from disk is the inode of the root directory. In most UNIX file systems, the root inode number is 2. Thus, to begin the process, the FS reads in the block that contains inode number 2 (the first inode block). Once the inode is read in, the FS can look inside of it to find pointers to data blocks, which contain the contents of the root directory. The FS will thus use these on-disk pointers to read through the directory, in this case looking for an entry for foo. By reading in one or more directory data blocks, it will find the entry for foo; once found, the FS will also have found the inode number of foo (say it is 44) which it will need next. The next step is to recursively traverse the pathname until the desired inode is found. In this example, the FS reads the block containing the inode of foo and then its directory data, finally finding the inode number of bar. The final step of open() is to read bar’s inode into memory; the FS then does a final permissions check, allocates a file descriptor for this process in the per-process open-file table, and returns it to the user.

### Writing

- Allocate a file
    - Read to inode bitmap (to find a free inode)
    - Write to inode bitmap (mark it allocated)
    - Write to new inode itself (to initialize it)
    - One to data of the directory (link high-level name of the file to its inode number)
    - One read and write to the directory inode and update it

- Modern systems, in contrast, employ a dynamic partitioning approach. Specifically, many modern operating systems integrate virtual memory pages and file system pages into a unified page cache

- **Caching on writes**: **buffering** 
    - Write buffering 
        - Pros 
            - Delay writes, file system can batch updates into smaller set of I/Os      
            - Schedule the subsequent I/O and thus increase performance 
            - Some writes are avoided altogether by delaying them (i.e. create and then delete) 
        - Cons
            - If system crash before update propagated, updates are lost 
            - Keeping writes in memory longer, performance can be improved by batching, scheduling, and even avoiding writes