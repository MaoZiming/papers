# Plan 9 from Bell Labs (1990)

Link: https://css.csail.mit.edu/6.824/2014/papers/plan9.pdf

Read: July 11th, 2024

Plan 9 is a distributed operating system, designed to make a network of heterogeneous and **geographically** separated computers function as a single system. In a typical Plan 9 installation, users work at terminals running the window system rio, and they access CPU servers which handle computation-intensive processes. Permanent data storage is provided by additional network hosts acting as file servers and archival storage

* People had grown weary of overloaded, bureaucratic timesharing machines and were eager to move to small, self-maintained systems, even if that meant a net loss in computing power. 

## Motivation
Small self-maintained machines made it easier for users to do work but made administration much more difficult. Instead Plan 9 uses network storage and CPU to do work. Per-process namespaces led to easier distributed systems and IPC.

## Key principles
* Resources are named and accessed like files in FS
* Standard protocol $P9$ for accessing these resources
* All resources in one single hierarchical namespace

**Key ideas**:
* **Everything is a file** (incl. hardware, window manager)

* The best was [Unix] use of the file system to coordinate naming of and access to resources, even those, such as devices, not traditionally treated as files. For Plan 9, we adopted this idea by designing a network-level protocol, called 9P, to enable machines to access files on remote systems. 

* The view of the system is built upon three principles. First, resources are named and accessed like files in a hierarchical file system. Second, there is a standard proto­col, called 9P, for accessing these resources. Third, the disjoint hierarchies provided by different services are joined together into a single private hierarchical file namespace.

* Central server with clients rendering windows. This centralizes administration while allowing users to customize their workspace.
* Integrated backups each day
* **Per process namespaces**
   * A single path name may refer to different resources for different processes.
* **A simple message-oriented file system protocol.**
   * Processes can offer their services to other processes by providing virtual files that appear in the other processes' namespace. The client process's input/output on such a file becomes *inter-process* communication between the two processes.
   * **Service-as-a-file**: **A process can offer a service by providing a *file* that other processes can read/write to.**
   * Sharing device across network can be accomplished by mounting the corresponding directory tree to the target machine.
* File systems are dumped into a write-once-read-many device for stable storage, every morning.
* $8\frac{1}{2}$: the windows system. What is unusual is how this is done: $8\frac{1}{2}$ is a file server, serving the files in `/dev` to the clients running in each win­ dow. Although every window looks the same to its client, each window has a distinct set of files in `/dev`. $8\frac{1}{2}$ multiplexes its clients access to the resources of the terminal by serving multiple sets of files. Each client is given a private name space with a different set of files that behave the same as in all other windows.
* Plan 9 has no super-user. Each server is responsible for maintaining its own secu­rity, usually permitting access only from the console, which is protected by a password.

* The intended style of use is to run interactive applications such as the window sys­tem and text editor on the terminal and to run computation- or file-intensive applica­tions on remote servers.
