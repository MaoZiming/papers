# Active Messages: A Mechanism for Integrated Communication and Computation (1992) 
**Key idea**: Include **code pointer in messages** which avoids buffering latency. Code is run immediately on server, thus avoiding as much queueing as possible. Handlers must be fast. Get communication onto network ASAP and off the network into computation ASAP.

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
