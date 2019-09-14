## Pipes and Fifos

### Pipe

A pipe is a mechanism for interprocess communication.

Data written to the pipe by one process can be read by another process.

Data is handled in a first-iin, first out order(FIFO) order.

!!! Pipe has no name; it is created for one use and both ends must be inherited from the single process which created the pipe.

### FIFO

FIFO is a special file similar to a pipe but instead of being anonymous, temporary connection, a FIFO has a name or names like any other file.

Processes open FIFOs by name to communicate through it.

### Pipe and FIFO communication

A pipe or FIFO has to be open at both ends simultaneously. 

If you read from a pipe or FIFO file that doesn't have any processes writing to it (perhaps because they have all closed the file, or exited), 
the read returns end-of-file. Writing to a pipe or FIFO that doesn't have a reading process is treated as an error condition; 
it generates a SIGPIPE signal, and fails with error code EPIPE if the signal is handled or blocked. 

Nether pipes nor FIFO special files allow file positioning. Both reading and writing operations happen sequentially; 
reading from the beginning of the file and writing at the end. 

### Creating a Pipe

```
#include unistd.h

// The pipe function creates a pipe and puts the file descriptors for the reading and writing ends of the pipe (respectively) into filedes[0] and filedes[1]. 
// An easy way to remember that the input end comes first is that file descriptor 0 is standard input, and file descriptor 1 is standard output. MIND BLOWN
int pipe (int filedes[2])
```

### Pipe Atomicity

Reading or writing pipe data is atomic if the size of data written is not greater than PIPE_BUF. 

This means that the data transfer seems to be an instantaneous unit, in that nothing else in the system can observe a state in which it is partially complete. 

Atomic I/O may not begin right away (it may need to wait for buffer space or for data), but once it does begin it finishes immediately.

Reading or writing a larger amount of data may not be atomic; for example, output data from other processes sharing the descriptor may be interspersed. 

Also, once PIPE_BUF characters have been written, further writes will block until some characters are read. 
