# Rethink the Sync (2006) 
The paper introduces external synchrony, a new model for file I/O. The model combines the reliability and simplicity of sync I/O with performance levels close to that of async I/O, being only 8% slower. 

The central problem addressed by the paper is the tension between **durability** and **performance** in file I/O. While sync I/O offers strong reliability and data durability guarantees, it comes at a performance cost and blocks the caller until the operation is complete. On the other hand, async I/O offers better performance but falls short on reliability, ordering, durability, and ease of programming.

## Insights 
The central insight is to define synchronous operations from the viewpoint of an **"external observer"** rather than just from the perspective of the application. This approach ensures that an external entity, be it a **user** or another system, can rely on the committed output without concerns of the output being predicated on uncommitted changes.

## Key Techniques
The application isn't blocked (like async I/O) but any output (e.x. network, sends to screen) depending on the file save operation is buffered until the data is committed, ensuring safety (like sync I/O).

**_External Sync I/O_**: 
* Returns control to the application before the data is committed to the disk but buffers all output that is causally dependent on the uncommitted modification.
* It only externalizes (sends to screen, network, etc.) the output after the modification commits
* Multiple file modifications are grouped together and committed as a single transaction, making the commits atomic.

_**Output triggered commits**_: data is only committed when there is an external output that depends on the uncommitted data

Example: If a process modifies data but produces no external output based on this modification, the system can delay the commitment of this data to optimize throughput. If output is generated that relies on this modification, the system triggers a commit to minimize latency.

_**Tracking causal dependencies**_: tracks dependencies between file modifications and external outputs. This ensures the external outputs are reliable and consistent with the sync I/O model. 
* marking process with dependencies: i.e. cannot externalize any output until some transaction commits
* if a marked process shares information with another process (IPC), another process inherits the same dependencies 
    
Example: xsyncfs adds modifications to a file system transaction and returns control to the process without waiting for the transaction to commit but taints it with a commit dependency. If the process tries to write to an external device, the output is buffered until all related disk transactions commit.

The new model builds upon a system called Speculator, initially targeted at improving network performance, to track and propagate causal dependencies.

## Limitations
Crash recovery is a problem in this paper, and there are many extra mechanisms to deal with catastrophic media failures and system crashes (via checkpoints and commits). 
