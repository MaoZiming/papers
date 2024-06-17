# Active Messages: A Mechanism for Integrated Communication and Computation (1992) 

Link: http://people.cs.uchicago.edu/~ftchong/290N-W12/isca92.pdf

Read: June 17th.

**Key idea**: Include **code pointer in messages** which avoids buffering latency. Code is run immediately on server, thus avoiding as much queueing as possible. Handlers must be fast. Get communication onto network ASAP and off the network into computation ASAP.

The underlying idea is simple: each message contains at its head the address of a user-level handler which is executed on message arrival with the message body as argument.

the effectiveness of the machine is low under traditional send/receive models due to poor overlap of communication and computation, and due to high communication overhead.

## Key techniques: active messages
1. **User-Level Handler**: This is essentially a code pointer that indicates a specific, small piece of code that needs to be executed when the message is received. The handler is shared and known across all machines in the network.

2. **Message Body**: This portion contains the actual data or arguments that the handler will use when it executes its code.

## Workflows 
1. _Message Sending_: wraps the data and the handler's address into an Active Message and sends 

2. _Asynchronous Operation_: sender does not wait for an ACK from the receiver and continues its own computation.

3. _Message Receiving_: receiver reads the Active Message, immediately executes the handler, passing in the provided arguments.

4. _Handler Execution_: The handler, being a lightweight piece of code, carries out its operation quickly and integrates the received data into the ongoing computation at the receiver's end. The idea is to **get the data "out of the network and into computation"** as quickly as possible.

## Use today 
* Microservices: benefit from the same principal, services being able to respond quickly
* Serverless computing: execute small functions or handlers immediately upon receiving specific messages or events, creating efficient, reactive systems 

### Problem

Communication between processors is slow, and speeding it up sacrifices cost / performance of the system 

### Goal

1. Minimize the communication overhead
2. Allow communication to overlap computation
3. Coordinate the two without sacrificing the cost / performance. 

### Message passing machine

- Message passing multiprocessors have unnecessarily high communication overhead

### Message driven machine

- Message driven machines demonstrate low communication overhead but poor processor cost / performance ratio

## Key Techniques

Active Messages - Integrate communication and computation 

- Messages consists of address of **a user-level handler** at the head, and the **arguments** to be passed as the body
- The **handler** gets the message out of the network and into ongoing computation as fast as possible
- No buffering
- A simple mechanism close to hardware that can be used to implement existing parallel programming paradigms

### Protocol

1. Sender sends a message to receiver 
    1. Async send while still computing 
2. Receiver pulls messages, integrates into computation through handler 
    1. Handler executes without blocking 
    2. Handler provides data to ongoing computation 
        1. Does not perform any computation itself
    3. Handler can only reply to sender, if necessary 

### Advantages

- Async communication
    - Non-blocking send / receive for overlap
- No buffering
    - Only buffering within network is needed
    - Software handles other necessary buffers
- Improved performance
    - Close association with network protocol
- Handlers are kept simple
    - Serve as an interface between network and computation
- Concern becomes overhead, not latency

### Weakness

- Restricted to SPMD model
  - Active Messages simply generalize the hardware functionality by allowing the sender to specify the address of the handler to be invoked on message arrival. Note that this relies on a uniform code image on all nodes, as is commonly used (the SPMD programming model). 
- Handler code is restricted
    - Can’t block and has to get the message out of the network as fast as possible
- Performance evaluation is not presented well in the paper
- Possible hardware support very speculative?

### Beyond

- Used in many MPI implementations at low-level transport layer

The network is viewed as a pipeline
○ The sender launches the message into the network and continues computation
○ The receiver gets notified or interrupted upon message arrival

The fundamental difference between the message driven model and Active Messages is where computa- tion-proper is performed: in the former, computation occurs in the message handlers whereas in the latter it is in the “background” and handlers only remove messages from the network transport buffers and integrate them into the computation. This difference significantly affects the nature of allocation and scheduling performed at message arrival.

Large message : If small messages are well supported, a large message should be viewed as a small one with a DMA transfer tacked-on.