### FIFO special files

created a FIFO special file in this way, any process can open it for reading or writing, in the same way as an ordinary file. 

However, it has to be open at both ends simultaneously before you can proceed to do any input or output operations on it. 

Opening a FIFO for reading normally blocks until some other process opens the same FIFO for writing, and vice versa. 

```
#include sys/stat.h

// Using O_NONBLOCK returns immediately but returns an error if opened for writing and no reader exists.
// what about opened for reading and no writer exists? read is blocking anyhow.
int mkfifo (const char *filename, mode_t mode)
```

Pitfall

```
// One immediately thinks of creating a nonblocking read fifo and use SIGIO.
// watchout for the following pitfall.

while (1) {
    fifo = open("fifo", O_RDONLY | O_NONBLOCK);
    fd_set read;
    FD_SET(fifo, &read);
    select(nfds, &read, NULL, NULL, NULL);
}

// whenever there is no writer to the FIFO, the read end will receive an EOF.
// Select system call will return on EOF and for every EOF you handle there will be a new EOF. This is the reason for the observed behavior.
// To avoid this open that FIFO for both reading and writing (O_RDWR).
```

