# Edge Computing


* Edge computing is a distributed computing paradigm in which
computation is largely or completely performed on distributed device node
known as smart devices or edge devices as opposed to primarily taking
place in a centralized cloud environment.
* Smart devices, cell towers, edge data centers, sensors, etc. 
* Reason for edge computing
  * Lower latency
  * Privacy
  * Reliability (network partition)
* VM encapsulation prevails. 

## Why VM encapsulation at the edge?
* Container: lightweight, fast
* VM:
  * Safety: protect infra from malicious software
  * Isolation: Hide mutually untrusting executions from one another
  * Transparency: run unmodified code
  * Deployability: compatibility 

## VM handoff
* More than just downtime, total completion time matters.
* WAN links have lower bandwidth
* Cutting all ties to the source hardware quickly. 
* Live VM migration assumes that the source machine can stay for longer.

* Assume a variety of base images available to use for eliminating unnecessary traffic. 

## To reduce the BW usage: 
* Use deduplication, compression, delta-encoding, etc.

## Reason for poor agility of live migration
* Total duration long
* Amount of data transferred is long. 

## Additional challenges

* In this paper, we use the range from 5 to 25 Mbps for our experiments, with the US average broadband Internet connectivity of 13 Mbps in 2015 falling in the middle of this range [2]. This is far from the 1–40 Gbps, low-latency and low-jitter connectivity that is typically available within data centers

* Container has lower resource usage -> Less burden on the cloudlet (edge devices). 

* The first is safety: protecting the integrity of cloudlet infrastructure from potentially malicious application software. The second is isolation: hid- ing the actions of mutually untrusting executions from each other on a multi-tenant cloudlet. The third is transparency: the ability to run unmodified application code without re- compiling or relinking. 


## Poor agility of live migration
Latency-sensitive applications. Post-copy approach may result in erratic application performance. 

## Base VMs.
It leverages the presence of a base VM at the destination: data blocks already present there do not have to be transferred. 

When a VM instance is launched, we first identify all the blocks that are different from the base VM. We cache this infor- mation in case of future launches of same image.

## VM handoff

* VM handoff pipelines the data reduction stages in order to fill the network as quickly as possible.

## Live migration in the cloud

* Load balancing for long-lived jobs. 
* Fault tolerance: move job away from flaky (but not yet
broken hardware)
* Data center is the right target


## Benefits of Migrating Virtual Machines Instead of Processes
* Avoids `residual dependencies’
  * old host handles local disk access, forwards network traffic
  * hard to move TCP state of an active connection for a process
* Allows separation of concern between users and operator of a datacenter or cluster


## Goals
* Minimize downtime (maximize availability)
* Keep the total migration time manageable
* Avoid disrupting active services by limiting impact of migration on both migratee and local network

## VM Memory Migration Options
* Push phase
* Stop-and-copy phase
* Pull phase

## Live VM Migration
* Writable working set. 

* Benefits:
  * Remove residual states
  * In-memory state can be transferred in a consistent and efficient fashion. This applies to kernel-internal state (e.g. the TCP control block for a currently active connection) as well as application-level state, even when this is shared between multiple cooperating processes.

* Minimize downtime
* Manageable total migration time
* Do not disrupt active services through resource contention. 


## Eschew the 'pull' approach - faults in missing pages across the network since this adds a residual dependency of arbitrarily long duration, as well as providing in general rather poor performance. 


Residual dependency cannot be solved in general by any process migration techniques. 


Bound the number of rounds of pre-copying, based on our analysis of the writable working set (WWS) behavior of typical server workloads. 


We address this issue by carefully controlling the network and CPU resources used by the migration process, thereby ensuring that it does not interfere excessively with active traffic or processing.

A migrating VM will include all protocol state (e.g. TCP PCB), and will carry its IP address with it.

Migrated VM can Generate ARP to redirect traffic to Host B

the migrating OS can keep its original Ethernet MAC address, relying on the network switch to detect its move to a new port. 

Using network-attached storag. NAS is uniformly accessible from all host machines in the cluster. 


Pre-migration, reservation, iteractive pre-copy, stop-and-copy, commitment, activation.


## The fundamental challenge of pre-copying is: how does one determine when it is time to stop the pre-copy phase because too much time and resource is being wasted?

Writable working set: identify the set of pages that will be written often and so should best be transferred via stop-and-copy. 

Use Xen's shadow page table to track dirtying statistics on all pages used by a particular executing operating system. 

The hottest pages place  alower bound in the amount of data transferred in the final stop-and-copy phases. 