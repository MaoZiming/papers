# Event-based Concurrency

- Popular in modern systems, like server-side frameworks (i.e. node.js)
- Problem it addresses
    - Managing concurrency correctly in multi-threaded applications can be challenging: dead lock, missing locks, etc.
    - The developer has little or no control over what is scheduled at a given moment in time

## Basic Idea: An Event Loop

- Basic idea: wait for an event to occur, then check what type of event it is, and do small amount of works it requires

## API: `select()` or `poll()`

- Check whether there is any incoming I/O that should be attended to
- I.e. a network application (web server) wishes to check whether any network packets have arrived, in order to service them
- Check whether descriptors can be read from as well as written to
    - Determine that a new packet has arrived and is in need of processing
    - Let the service know when it is ok to reply
- Timeout = NULL: block indefinitely until some descriptors is ready
    - Common: set it to 0, then use the call to `select()` to return immediately
- Poll() system is pretty similar
- Use non-blocking event loop

## Blocking v.s. Non-blocking Interfaces
- Blocking (or synchronous) interfaces do all of their work before returning to the caller
- Non-blocking (or asynchronous) interfaces begin some work but return immediately, thus letting whatever work that needs to be done get done in the background.
- Solution: Async I/O
    - `struct aiocb` or AIO control block
    - Application fill in this structure, then issue async call to read the file `int aio_read(struct aiocb *aiocbp);`
        - Try issuing I/O, if successful, return right away
    - `int aio_error(const struct aiocb *aiocbp);`
        - Checks whether the request referred to by `aiocbp` has completed
        - Periodically poll the system
        - Interrupt (signals) to inform application when AIO completes, removing needs to repeatedly ask the system
        - Q: how does this compare to the mechanism of thread-based server?

