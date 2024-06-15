# Proportional Share 
Proportional share scheduler sometimes is referred to as a **fair-share scheduler**. It is based around a simple concept: instead of optimizing for turnaround or response time, a scheduler might instead try to guarantee that each job obtain a certain percentage of CPU time. 

The generla problem with fair-share scheduler is that, these approaches do not particularly mesh well with I/O, i.e. jobs that perform I/O occasionally may not get their fair share of CPU. Also ticket assignment or priority assignment is an open hard problem. 

But many domains this is not major concern, like in a virtualized data center (i.e. in clouds), where you like to assign 1/4 of CPU cycles to Windows VM and rest to Linux, then proportional sharing is simple and effective. Also you can proportionally share memory in VMWare's ESX server. 

## Lottery Scheduling 
Use **tickets** to represent the share of a resource that a process should have. The scheduler picks a winning tkcet (among total # of tickets), and the winning tickets determine which process should run. 

<img width="729" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/2634bdfb-5c6e-441f-9dac-8379f3d96e93">


The benefits of this approach is that it is lightweight and easy to implement, there is no global state. The cons are that the scheduling is non-deterministic, and ticket assignment is a difficult problem. 

## Stride Scheduling 
Deterministic fair-share scheduler. Each job in the system has a **stride**, which is inverse in proportion to the number of tickets it has. We can compute the stride of each by dividing some large number by the number of tickets each process has been assigned. 

At any given time, pick the process to run that has the lowest pass value so far. When run a process, increment its **pass** counter by stride. Pros is deterministic, but cons is need to maintain global state per process. 

<img width="464" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/4d0a6bc7-513e-4e7f-b285-86b265d1becb">

## Linux: CFS
Completely Fair Scheduler (cfs) implements fair-share scheduling, but does so in a highly efficient and scalable manner. 

The basic idea of CFS is to fairly divide a CPU evenly among all competing process, with the use of **virtual runtime** (`vruntime`), which increases in proportion with physical time. CFS pick the process with the lowest `vruntime` to run next. 

There are several parameters to determine how long the time slice is (i.e. `sched_latency`, `min_granularity`), enable control over process priority (`nice` and weight) and so on. It uses red-black tree to keep runnable processes to enable efficient scheduling. To avoid starvation, CFS sets the `vruntime` of that job to minimum value found in the tree. 

<img width="496" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/656f8ac7-8202-41fc-9931-d17b1a45d99a">

