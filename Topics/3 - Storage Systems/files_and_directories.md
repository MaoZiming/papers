# Files and Directories

## Intro

- **Persistent storage**: e.x. classic hard disk drive or solid-state storage device
    - Store information permanently
    - Unlike memory, where contents are lost when there is a power lost

## What is a file

- **File:** array of persistent bytes that can be read / written
- Low-level name of a file is called i-node number. 
- **File system** consists of many files
    - Refers to collection of files
    - Also refers to part of OS (e.x. ext3, ext4, NTFS, etc.) that manages those files
- Files need names to access correct one, and there are three types of names
    - Unique id: inode number
    - Path
    - File descriptor
- Directory: like a file, but contains a list of (user-readable name, low-level name)
  - It also has a low-level name (e.g. inode number)
  - e.g.  (“foo”, “10”)

## Paths

- String names are friendlier than number names
- File system still interacts with inode numbers
- **Directory tree** instead of single root directory
- **File name** needs to be unique within a directory
- Store **path-to-inode mappings** in each directory
    - Reading for getting final inode is called “traversal”

## File Descriptor (FD)

- `open()` returns: a file descriptor. A file descriptor is just an integer, private per process, and is used in UNIX systems to access files; thus, once a file is opened, you use the file de- scriptor to read or write the file, assuming you have permission to do so.
  - File descriptor is as a pointer to an object of type file
- Idea
    - Do expensive traversal once (open file)
    - Store inode in descriptor object (kept in memory)
    - Do reads / writes via descriptor, which **tracks offset**
- Each process: file descriptor table contains pointers to **open file descriptors**
- Integer used for file I/O are indexes to this table
    - stdin: 0, stdout: 1, stderr: 2
  - The subsequent file descriptors start from 3. 
```c
struct proc {
    ...
    struct file *ofile[NOFILE];
    ...
}
```
- the `file` array is indexed by the file descriptor. 

## Interface
```c
int fd = open(char *path, int flags, mode_t mode);
read(int fd, void *buf, size_t nbyte);
write(int fd, void *buf, size_t nbyte);
close(int fd);
```
- If an attempted `read()` past the end of the file returns zero, it indicates to the process that it has read the file in its entirety.
- Advantages
    - String names
    - Hierarchical
    - Traverse once
    - Offsets precisely defined (especially when multiple processes are sharing)

## Reading And Writing, But Not Sequentially

```c
    off_t lseek(int fildes, off_t offset, int whence);
```
Values that `whence` can take: 
```
  If whence is SEEK_SET, the offset is set to offset bytes.
  If whence is SEEK_CUR, sthe offset is set to its current
    location plus offset bytes.
  If whence is SEEK_END, the offset is set to the size of
    the file plus offset bytes.
```
OS tracks a current `offset` for each file that it opens. 
`read` and `write` also changes the current `offset`.

In `struct file`: 
```c
struct file {
  int ref;
  char readable;
  char writable;
  struct inode *ip;
  uint off;
};
```

- Open file table. The xv6 kernel just keeps these as an array, with one lock for the entire table.
```c
struct {
  struct spinlock lock;
  struct file file[NFILE];
} ftable;
```

## Sharing files

- `fork`: increments the reference count. Parent and child shares the file.
- For example, if you create a number of processes that are cooperatively working on a task, they can write to the same output file without any extra coordination.
- ![alt text](image-6.png)
- The `dup()` call allows a process to create a new file descriptor that refers to the same underlying open file as an existing descriptor.

## FSYNC: communicating requirements

- Most times when a program calls write(), it is just telling the file system: please write this data to persistent storage, at some point in the future.
- File system keeps newly written data in memory for a while
    - Write buffering improves performance
- If system crashes before buffers are flushed, then lose data
- `fsync(int fd)`
    - Forces buffer to flush to disk
    - Tells disk to flush its write cache
    - Makes data durable
- In the UNIX world, the interface provided to applications is known as `fsync(int fd)`. When a process calls `fsync()` for a particular file descriptor, the file system responds by forcing all dirty (i.e., not yet written) data to disk, for the file referred to by the specified file descriptor. The `fsync()` routine returns once all of these writes are complete.

## Renaming files

- One interesting guarantee provided by the rename() call is that it is (usually) implemented as an atomic call with respect to system crashes; if the system crashes during the renaming, the file will either be named the old name or the new name, and no odd in-between state can arise.

## Getting information about a file

- `fstat`: contain information stored in places such as inode.
- Inode is a persistent data structure, but are cached in memory for performance reasons.

## Reading directories
- `opendir()`, `readdir()`, and `closedir()`
- `struct dirent` data structure
```c
struct dirent {
  char d_name[256]; // filename
  ino_t d_ino; // inode number
  off_t d_off; // offset to the next dirent
  unsigned short d_reclen; // length of this record
  unsigned char  d_type; // type of file
};
```

## Deleting Files

- There is no system call for deleting files!
  - It just calls `unlink()`.
- Inode (and associate file) is **garbage collected** when there are no references
    - Paths are deleted when `unlink()` is called
    - FDs are deleted when `close()` or process quits
- `link`: an old pathname and a new one; when you “link” a new file name to an old one, you essentially create another way to refer to the same file.
- The way `link()` works is that it simply creates another name in the directory you are creating the link to, and refers it to the same inode num- ber (i.e., low-level name) of the original file. The file is not copied in any way;
- Hard link
  - `ln` command
    - Hard links are somewhat limited: you can’t create one to a directory (for fear that you will create a cycle in the directory tree); you can’t hard link to files in other disk partitions (because inode numbers are only unique within a particular file system, not across file systems); etc. Thus, a new type of link called the symbolic link was created
    - A hard link is essentially another name for an existing file on disk
        - All hard links to a file point to the same inode, and the same data blocks
- What `unlink` does:
  - First, you are making a structure (the inode) that will track virtually all relevant infor- mation about the file, including its size, where its blocks are on disk, and so forth. Second, you are linking a human-readable name to that file, and putting that link into a directory.
  - The reason this works is because when the file system unlinks file, it checks a reference count within the inode number. This reference count (sometimes called the link count) allows the file system to track how many different file names have been linked to this particular inode. 
    - only when the reference count reaches zero does the file system also free the inode and related data blocks, and thus truly “delete” the file.
- Soft link (symbolic link)
    - `ln -s`. 
    - Difference
        - A symbolic link is actually a file itself, of a different type
        - A symbolic link is a separate file that contains a reference to another file or directory in the form of an absolute or relative path to target
            - Have their own inode numbers
    - Potential problem
        - Dangling reference
        - Quite unlike hard links, removing the original file named file causes the link to point to a pathname that no longer exists

## Permission Bits and ACL

- UNIX: **permission bits**
    - Three groupings of permissions
        - What the **owner** of the file can do to it
        - What someone in a **group** can do to the file
        - What anyone (i.e. **other**) can do

- **Access control list (ACL)**
    - More complicated control to represent exactly who can access a given resource
    - Enable a user to create a specific list of who can and cannot read a set of files
        - V.s. limited owner / group / everyone model of permission bits
    - E.x. AFS

## Making and Mounting FS

- Make a file system: `mkfs`
    - Give the tool, as input, a device (e.g. a disk partition), a file system type (e.g., ext3)
    - Write an empty file system, start with root directory, onto that disk partition
- To make a FS accessible within the uniform file-system tree: `mount`
    - Idea: take existing directory as a **target mount point** and essentially paste a new FS onto the directory tree at that point