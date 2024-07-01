# Hard Disk Drives

### Basic geometry

1. **Platter**: circular hard surface on which data is stored persistently by including magnetic changes to it 
    1. A disk may have one or more platters
    2. Each platter has 2 sides, each of which is called a **surface** 
    3. Made of some hard material (i.e. aluminum), then coated with thin magnetic layer that enables the drive to persistently store bits even when the drive is powered off 
2. **Spindle:** the platters are all bound together around the spindle 
    1. Connect to a motor that spins the platters around (while the drive is powered on) at a fixed rate 
    2. Rate: **rotations per minimute (RPM)** 
    3. Typical value: 7,200 - 14,000 RPM 
    4. 10,000 RPM —> single rotation is 6 ms
3. **Track:** data is encoded on each surface in concentric circles of sectors, one such concentric circle is a track 
    1. Stack of tracks (across platters): cylinder
4. **Disk head:** the process of read and write is accomplished by disk head 
    1. One such head per surface of the drive 
    2. Attached to a single **disk arm**, which moves across the surface to position the head over the desired track

### Simple disk drive

- Single track latency = **rotational delay**
- Multiple track = **seek time**
    - Move the disk arm to the correct track
    - Some phases: acceleration phase (disk arm gets moving), coasting (arm is moving at full speed), deceleration (arm slows down), settling (positioned at the right track)
    - Settling alone can take 0.5-2 ms
    - Entire seeks often takes 4-10 ms
- Data read or written from / to the surface = **transfer time**
    - Pretty fast, depending on RPM and sector density
    - 100+ MB/s is typical for maximum transfer rate
- Time to R/W = seek + rotation + transfer time

### Other improvement

- Track skew: account for rotation and seek happening at the same time
    - Sequential reads across track boundaries
- Zones
    - ZBR (zoned bit recording): more sectors on outer tracks
    - Improve density of the data stored
- Cache
    - Drives may cache both reads and writes (in addition to OS cache)
    - Read? avoid doing disk seek if sector is in cache
    - Write? Ack writes before it is persisted
    - Disks contain internal memory (2-16MB) used as cache
    - Read-ahead: “Track buffer”
        - Read contents of entire track into memory during rotational delay
    - Write caching with volatile memory
        - Immediately reporting: claim written to disk when not
        - Data could be lost on power failure
    - Tagged command queueing
        - Have multiple outstanding requests to disk
        - Disk can reorder (schedule) requests for better performance

### 1) FIFO

First in first out, however seek + rotation overhead can be significant 

### 2) SSTF (Shortest Seek Time First)

Always choose request that requires least seek time (approximate total time with seek time) 

- Greedy algorithm (look for best NEXT decision)
- How to implement in OS?
    - Sort sector numbers
- Disadvantages?
    - Starvation of some sectors far away from the current location

### 3) Elevator (a.k.a. SCAN or C-SCAN)

Sweep back and forth, from one end of disk other, serving requests as pass that cylinder 

- Sorts by cylinder number, ignores rotation delays
- C-SCAN (circular scan): only sweep in one direction
    - A bit more fair to inner and outer tracks, as pure back-and-forth SCAN favors the middle track (i.e. middle track passed twice before coming back to outer track again)
- Also elevator algorithm
    - Elevator: 10 —> 1, somebody got on 3, and press 4, then the elevator will not go up to 4 because it is closer!
- Cons
    - Ignore rotation

### 4) SPTF (Shortest Positioning Time First) or SATF (Shortest Access Time First)

- Greedy algorithm taking into account both seek and rotation costs
- This is typically done inside the disk (OS has no idea about the zone, layout, etc.)

### Other stuff

- Disk scheduler would also perform **I/O merging**
- How long should the system wait before issuing an I/O to disk?
    - **Work-conserving**: immediately issue request to the disk, always try to do work if there’s work to be done
    - **Anticipatory disk scheduling**: wait a bit


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