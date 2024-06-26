# Input/Output (I/O) and System Performance
- Conceptual understanding of buses, controllers, interrupts, polling, programmed I/O (PIO), and Direct Memory Access (DMA)
- Addition: I/O schedulers

## Three Main Question

1. **How to avoid the cost of polling** 
    1. How can OS check device status without frequent polling, and lower the CPU overhead required to manage the device? 
    2. Tradeoff: **interrupt** v.s **polling** 
2. **How to lower PIO overheads**
    1. With PIO, CPU spends too much time moving data to and from device by hands 
    2. How can we offload this work and allow CPU to be more effectively utilized? 
    3. Tradeoff: **PIO** v.s **DMA** 
3. **How to communicate with devices?**
    1. How should hardware communicate with a device? Should there be explicit instructions? 
    2. Tradeoff: **Explicit I/O instructions**, **memory-mapped I/O** 

## Buses, Controllers
<img width="490" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/93f45763-6547-49f6-b080-e7a9c8f10b30">

General architecture:

- Single CPU attached to main memory with **memory bus** or interconnect
- Some devices are connected to the system via a general **I/O bus** (e.g. PCI)
- **Peripheral bus** (e.g. USB, SCSI, SATA) connects **slow devices** to the system, including disks, mice, and keyboards
- Why hierarchy?
    - Physics, and cost
    - Components demand high performance (e.g. graphic card) are nearer the CPU
    - Low performance components are further away
      
<img width="420" alt="image" src="https://github.com/lynnliu030/os-prelim/assets/39693493/ae3150d7-e322-489b-86c5-f6ff7068e0c4">


Registers 
1. Status register: read to see current status of the device 
2. Command register: tell the device to perform a certain task
3. Data register: pass data or get data to / from the device 

## PIO protocol: Basic Steps of OS of doing I/O

1. Polling the device 
2. Sending data down to data register 
3. Writing a command to command register, let the device know that data is present and it should begin working
4. Waiting until the device to finish by polling it from a loop 

## Interrupts v.s. Polling

- Polling
    - OS check devices status with frequent polling
    - Pros: high CPU overhead
- Interrupts
    - When device finished operation, raise a hardware interrupt
    - CPU jump into the OS at predetermined **interrupt service routine (ISR)** or, **interrupt handler**
    - The handler (i.e. a piece of OS code) finishes the request and wakes the process waiting for I/O
- Trade-offs
    - If device finishes quickly or in networks where there is huge stream of incoming packets
        - then interrupt overhead (context switching) is expensive, OS can even livelock, better to poll

## PIO v.s. DMA

- **PIO (programed I/O)**: CPU directly tells device what the data is
    - too much time moving data to and from device by hand
- Solution: **direct memory access (DMA)**
    - DMA: a very specific device within a system that can orchestrate transfer between devices and main memory without much CPU intervention
- To transfer data to device
    - OS program the DMA engine by telling it where data lives in memory, how much data to copy, and which device to send it to
    - OS is done with transfer and proceed with other work
    - DMA completes —> raise an interrupt, OS knows it complete
