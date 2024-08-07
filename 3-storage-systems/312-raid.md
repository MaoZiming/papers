# Redundant Array of Inexpensive Disks

Link: https://www.cs.cmu.edu/~garth/RAIDpaper/Patterson88.pdf

Read: July 12th, 2024

* Building I/O Systems as arrays of expensive disks. 

However:
* Without fault tolerance, large arrays of expensive disks are too unreliable to be useful.

Hence: Redundant Array of Inexpensive Disks vs. Single large expensive disk. 

**MTTR**: Mean time to repair a failed disk. 

## Summary 
* RAID: a faster, larger, and more reliable disk system
    * One logical disk built from many physical disk
    * Workloads: R/W, fail-stop fault model
    * Can be either in software or hardware. 
* RAID-0: data striping
    * Optimize for capacity, no redundancy
    * ![RAID-0](https://www.stationx.net/wp-content/uploads/2024/02/RAID-0-vs-RAID-1.png)
* RAID-1: data mirroring
    * Keep mirrored copies of the data
    * Writes slow down (have to wait for both)
    * Read: split between two disks. Read improves. 
* RAID-2: Hamming Code for ECC (Error correcting code)
  * Assumes: disk can fail by returning bogus data. 
  * Identify the disk has failed and recover. 
  * Both reads and writes have to be from whole disk.
    * Read has to check for error correction to detect bogus data. 
  * Adding check disk for error correction.
  * ![alt text](images/312-raid/raid-2.png)
* For a group size of 10 data disks (G) we need 4 check disks in total, and if G=25, then C=* 
* RAID-3: 
  * **Don't have to identify disk failures. Only need to correct!**
  * Most disk controllers an already detect if a disk failed. 
  * RAID-3 relies on that. 
  * Check bit. Use the check bit to determine the lost bit. 
  * **Byte-level striping** with a dedicated parity disk. 
  * Cannot use more than * 
  * bytes distributed across the disks. Reading blocks require touching all the disks. 
  * Reducing check disk to 1 per group. 
  * ![alt text](images/312-raid/raid-3.png)
    * Performance: Byte-level striping can lead to high data transfer rates for large files because multiple disks are read/written simultaneously. However, RAID 3's performance may suffer with multiple simultaneous small I/O operations due to the need to read from all disks to access any piece of data.

* RAID-4: use a parity disk. (a single check disk)
  * **block-level striping**
    * All parity block stored one disk.
    * read: only read block once. 
    * modify: two disks.
    * Parity: allow reconstruction of missing or corrupted data
    * Small write problem: parity disk is the bottleneck
    * ![alt text](images/312-raid/raid-4.png) 
    * Block-level striping improves random read performance because individual blocks of data can be accessed from separate disks without needing to read from all disks. However, the dedicated parity disk can become a bottleneck during write operations because every write involves updating the parity information.
* RAID-5: rotating parity
  * Block-level stripipng with distributed parity. 
    * Random write improves!
    * ![RAID-5](https://www.stationx.net/wp-content/uploads/2024/02/What-is-RAID-5.png)


## Metrics

Assume:

* $N =$  number of disks
* $C =$  capacity of 1 disk
* $S =$  sequential throughput of 1 disk
* $R =$  random throughput of 1 disk
* $D =$  latency of one small I/O operati

## RAID-0: Striping

  * Optimize for capacity, no redundancy
  * Stripes and chunk size
      * Stripe: box in the same row
      * Chunk size: # of blocks placed on one disk before moving to the next
          * Smaller chunk size: many files striped across many disks
              * Increase parallelism of read and write
              * But positioning time to access blocks increases
          * Larger chunk size
              * Reduce intra-file parallelism
              * But reduce positioning time
  * Capacity: $N \times C$
  * Reliability: $0$
      * No redundancy or resiliency
  * Performance
      * Latency (random): $D$
          * Choose a disk where the block is located, read latency the same as that disk
      * Throughput (sequential, random): ($N \times S$, $N \times R$)
          * Can send requests to disk in parallel

## RAID-1: Mirroring 

* Capacity: $\frac{N}{2} * C$
* Reliability: $1$
    * At least, some times more than 1 failure is okay (up to $\frac{N}{2}$ if lucky)
* Performance
    * Latency: $D$
    * Throughput
        * Random reads: $N * R$
        * Random writes: $\frac{N}{2} * R$
            * Requires **two physical write** ($\frac{N}{2}$) to complete before it is done
            * Two writes can be done in parallel, but logical write must wait for both physical write to complete
        * Sequential writes: $\frac{N}{2} * S$
        * Sequential reads: $\frac{N}{2} * S$
            * Rotating over skipped block, not deliver useful bandwidth to the client

## RAID-4
* Use **parity** disk
    * Can use XOR to compute: fast to implement, prevent overflow, easy to invert

* **Additive Parity**
    * Read C0, C1, C2, C3 in parallel
        * Assign C3 a new value
        * Compute the new parity
    * Write C3_new, P_new in parallel
    * Problem: scales with the number of disks, larger RAIDs require a higher number of reads to compute parity

* Random write performance: $R/ 2$
    * For each write, RAID has to perform 4 physical I/Os (two reads and two writes). Imagine lots of writes are submitted to RAID
    * **Small-write problem**: Parity disk is the bottleneck under this type of workload
        * Even though the data disk can be accessed in parallel, the parity disk prevents any parallelism from materializing
        * All writes to the system will be serialized because of the parity disk
        * Parity disk performs two I/Os per logical operation: one read, one write

Analysis

* Capacity: $(N-1) * C$
* Reliability: $1$
* Performance
    * Latency
        * Read: $D$
        * Write: $2D$
            * Read D4 parity, compute updated parity
            * Write back D3, D4 parity updated in parallel
    * Throughput
        * Random reads: $(N-1)*R$
        * Sequential writes: $(N-1) * S$
        * Sequential reads: $(N-1) * S$
        * Random writes: $\frac{N}{4} * R$
          * Read old data, old partiy, write new data, new parity. 

## RAID-5

* Rotate parity across different disks
* Random write?
    * Assume subtractive parity
    * To update a particular value, read this value and the parity value, then write back
    * Originally best that random write can do is $N * R$
    * Now, we can issue 4 request for every logical request I need to satisfy, but this can spread across the disk

### Analysis

* Capacity: $(N-1) * C$
* Reliability: $1$
* Performance
    * Latency
        * Read: $D$
        * Write: $2D$
            * Read D4 parity, compute updated parity
            * Write back D3, D4 parity updated in parallel
    * Throughput
        * Random reads: $N * R$
            * Improved!
            * Can actually use the throughput of all disks to get random access, assume that random reads spread out across different stripes
        * Sequential writes: $(N-1) * S$
        * Sequential reads: $(N-1) * S$
        * Random writes: $\frac{N}{4} * R$
            * Each RAID-5 write still generate 4 total I/O operation, which is simply the cost of using parity-based RAID
