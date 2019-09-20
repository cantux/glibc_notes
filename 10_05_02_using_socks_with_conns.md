### Using Sockets with Connections

Making a connection is asymmetric; one side (the client) acts to request a connection, while the other side (the server) makes a socket and waits for the connection request

#### Connect

Client makes a **connection** while server wats for and accepts the connection.

```
#include "sys/socket.h"

// Normally, connect waits until the server responds to the request before it returns. 
// You can set nonblocking mode on the socket socket to make connect return immediately without waiting for the response: O_NONBLOCK + select??
// sockaddr -> AF_INET, AF_LOCAL
int connect (int socket, struct sockaddr *addr, socklen_t length)
```

#### Listening

Server must use **listen** function to enable connection requests. Server then accepts connections with using **accept**(shocker) function.

**select** function reports when the socket has a connection ready to be accepted. See [Waiting for I/O](07_04_waiting_for_io.md).

Listen function is not allowed for sockets using connectionless communication styles. (SOCK_DGRAM - including socketpair)

You can write a network server that does not even start running until a connection to it is requested. See (Inetd Servers)[10_05_04_inetd.md]. 



```
// The argument n specifies the length of the queue for pending connections. When the queue fills, new clients attempting to connect fail with ECONNREFUSED until the server calls accept to accept a connection from the queue
int listen (int socket, int n)
```

#### Accepting

A socket that has been established as a server can accept connection requests from multiple clients.

The server's original socket does not become part of the connection; instead, accept makes a new socket which participates in the connection. accept returns the descriptor for this socket. 

The number of pending connection requests on a server socket is finite.
If connection requests arrive from clients faster than the server can act upon them, the queue can fill up and additional requests are refused with an ECONNREFUSED error. 
You can specify the maximum length of this queue as an argument to the listen function

```
// Waits unless O_NONBLOCK set
// Returns a new socket 
// Returns client name with addr, length_ptr
// This is a cancellation point
int accept (int socket, struct sockaddr *addr, socklen_t *length_ptr)
```

#### Who is connected

See [Socket Addresses](10_01_socket_addresses.md)
```
int getpeername (int socket, struct sockaddr *addr, socklen_t *length-ptr)
```


