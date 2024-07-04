# Active Messages: A Mechanism for Integrated Communication and Computation (1992)

Link: http://people.cs.uchicago.edu/~ftchong/290N-W12/isca92.pdf

Read: June 17th.

* Upcall and LRPC focus on local communication (different aspects, vertically and horizontally), and active messages focus on remote communication.
* The **effectiveness** of the machine is low under traditional send/receive models due to **poor overlap of communication and computation**, and due to high communication overhead.
  * Before: the message driven model and Active Messages is where computation-proper is performed: in the former, computation occurs in the message handlers whereas in the latter it is in the “background” and handlers only remove messages from the network transport buffers and integrate them into the computation.
* **Key idea**: Include **code pointer in messages** which avoids buffering latency. Code is run immediately on server, thus avoiding as much queueing as possible. Handlers must be fast. Get communication onto network ASAP and off the network into computation ASAP.
* The underlying idea is simple: each message contains at its head the address of a user-level handler which is executed on message arrival with the message body as argument.

## Key techniques: active messages

1. **User-Level Handler**: This is essentially a code pointer that indicates a specific, small piece of code that needs to be executed when the message is received. **The handler is shared and known across all machines in the network.**
2. **Message Body**: This portion contains the actual data or arguments that the handler will use when it executes its code.

## Workflows

1. _Message Sending_: wraps the data and the handler's address into an Active Message and sends
2. _Asynchronous Operation_: sender does not wait for an ACK from the receiver and continues its own computation.
3. _Message Receiving_: receiver reads the Active Message, **immediately executes the handler, passing in the provided arguments.**
4. _Handler Execution_: The handler, being a lightweight piece of code, carries out its operation quickly and integrates the received data into the ongoing computation at the receiver's end. The idea is to **get the data "out of the network and into computation"** as quickly as possible.

## Use today

* **Microservices**: benefit from the same principal, services being able to respond quickly
* **Serverless computing**: execute small functions or handlers immediately upon receiving specific messages or events, creating efficient, reactive systems
* Used in many MPI implementations at low-level transport layer

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

- Restricted to **SPMD** model
  - Active Messages simply generalize the hardware functionality by allowing the sender to specify the address of the handler to be invoked on message arrival. **Note that this relies on a uniform code image on all nodes, as is commonly used (the SPMD programming model).**
- Handler code is restricted
  - Can’t block and has to get the message out of the network as fast as possible
- Performance evaluation is not presented well in the paper
- Possible hardware support very speculative

