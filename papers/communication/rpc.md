# RPC Explain 
## Step-by-step Working of RPC with Direct Memory Execution Mode
1. **Local Processing** (In User Space)
* **Marshalling**: The client program initiates the RPC by calling the stub procedure, passing the necessary parameters. The client stub then **marshals (packages) these parameters** into a format suitable for transmission over a network.
2. **Communication to Kernel Space**
* System Call: After marshalling, a system call is made to pass the data from user space to kernel space.
* Data is copied from the user space memory to the kernel space memory.
3. **Transmission Over Network** (From Kernel Space)
* Network IO: The kernel then transmits this data packet over the network to the remote system.
4. **Reception at Remote System** (In Kernel Space)
* Data packet is first received in the kernel space of the remote system.
5. **Communication to User Space at Remote System**
* system Call: Through a system call, the data packet is moved from kernel space to user space, involving another data copy operation.
6. **Remote Processing** (In User Space)
    * **Unmarshalling**: the server stub unmarshals (unpacks) the parameters from the data packet.
    * **Execution**: The actual procedure on the server-side is then executed using the unpacked parameters.
7. **Sending the Response**
  * Marshalling the Response: After execution, the result (if any) needs to be sent back to the client. The server stub marshals the result into a data packet.
  * System Call to Kernel Space: The data packet is passed to the kernel space through a system call, involving a data copy operation.
  * Transmission Over Network: The kernel sends this data packet over the network to the client system.
8. **Receiving the Response** at Client
  * Reception in Kernel Space: The data packet with the result reaches the kernel space of the client system.
  * System Call to User Space: Through a system call, the data packet is moved to the user space, involving a data copy operation.
9. **Final Unmarshalling** (In User Space)

## Data Copy 
1. Copy from client user space to client kernel space (while sending the request).
2. Copy from server kernel space to server user space (to process the request).
3. Copy from server user space to server kernel space (to send the response).
4. Copy from client kernel space to client user space (to unmarshal the response).
