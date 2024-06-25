# Distributed Systems

- UDP includes a checksum to detect some forms of packet corruption.

- Distributed shared memory (DSM) systems enable processes on different machines to share a large, virtual address space
  - This approach is not widely in use today for a number of reasons. The largest problem for DSM is how it handles failure.

## RPC

### Stub Generator
  - The input to such a compiler is simply the set of calls a server wishes to export to clients. Conceptually, it could be something as simple as this:

```c
interface {
  int func1(int arg1);
  int func2(int arg1, int arg2);
};
```
  - The stub generator takes an interface like this and generates a few dif- ferent pieces of code. For the client, a client stub is generated, which contains each of the functions specified in the interface; a client program wishing to use this RPC service would link with this client stub and call into it in order to make RPCs.
  - To the client, the code just appears as a function call (e.g., the client calls func1(x)); internally, the code in the client stub for func1() does this:
  - Create a message buffer. A message buffer is usually just a con- tiguous array of bytes of some size.
  - Pack the needed information into the message buffer. The process of putting all of this information into a single contiguous buffer is sometimes referred to as the marshaling of arguments or the serialization of the message.
  - Send the message to the destination RPC server.
  - Wait for the reply. Because function calls are usually synchronous, the call will wait for its completion.
  - Unpack return code and other arguments. This step is also known as unmarshaling or deserialization.
  - Return to the caller. Finally, just return from the client stub back into the client code.

### Run-time library

- Many RPC packages are built on top of unreliable com- munication layers, such as UDP.
- The run-time must also handle procedure calls with large arguments, larger than what can fit into a single packet. Some lower-level network protocols provide such sender-side fragmentation (of larger packets into a set of smaller ones) and receiver-side reassembly (of smaller parts into one larger logical whole); if not, the RPC run-time may have to implement such functionality itself.


One issue that many systems handle is that of byte ordering. As you may know, some machines store values in what is known as big endian ordering, whereas others use little endian ordering. Big endian stores bytes (say, of an integer) from most significant to least significant bits, much like Arabic numerals; little endian does the opposite. Both are equally valid ways of storing numeric information; the question here is how to communicate between machines of different endianness.