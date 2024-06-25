# Fast File System (1984) 
The old UNIX file structure consists of large chunks of super block, inode region, and data region: data was spread all over the place ignoring positioning cost, data blocks are often very far away from the inode. 

Problems: all about performance 
1) _The old FS only deliver 2% of the overall bandwidth_
2) Fragmented FS
3) Original block size was too small, positioning overhead is huge

FFS is the first disk-aware file system that improves performance by changing FS structures and allocations policies. 

## Goal: performance 
Key optimizations:
* Improve locality of the data structures
  *   Keep related stuff together
  *   Use disk block _groups_ to keep files within consecutive blocks 
  *   File inodes allocate in the same group with directory
  *   Directory inodes allocated in new group with fewer used inodes than average group
  *   First data blocks allocated near inodes
  *   Other data blocks allocate near data blocks
* Reduce fragmentation of disk blocks by allocating contiguously from the bit maps
* Increase block size to inmprove performance
  *   To mitigate internal fragmentation, also uses sub-blocks
 
## Secondary goal: usability    
* Long file names
* Atomic rename
* Symbolic links
  *  hard link limited, can't point to directories, can't point to files across volumes    
