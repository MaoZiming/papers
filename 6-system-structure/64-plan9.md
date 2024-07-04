# Plan 9 from Bell Labs (1990)

Plan 9 is a distributed operating system, designed to make a network of heterogeneous and **geographically** separated computers function as a single system. In a typical Plan 9 installation, users work at terminals running the window system rio, and they access CPU servers which handle computation-intensive processes. Permanent data storage is provided by additional network hosts acting as file servers and archival storage

**Key ideas**:
1. **Everything is a file** (incl. hardware, window manager)
2. Central server with clients rendering windows. This centralizes administration while allowing users to customize their workspace.
3. Integrated backups each day
4. **Per process namespaces**
   1. A single path name may refer to different resources for different processes.
5. **A simple message-oriented file system protocol.**
   1. processes can offer their services to other processes by providing virtual files that appear in the other processes' namespace. The client process's input/output on such a file becomes *inter-process* communication between the two processes.
   2. Service-as-a-file: A process can offer a service by providing a *file* that other processes can read/write to.
   3. Sharing device across network can be accomplished by mounting the corresponding directory tree to the target machine.

## Motivation
Small self-maintained machines made it easier for users to do work but made administration much more difficult. Instead Plan 9 uses network storage and CPU to do work. Per-process namespaces led to easier distributed systems and IPC.

## Key principles
1. Resources are named and accessed like files in FS
2. Standard protocol for accessing these resources
3. All resources in one single hierarchical namespace
