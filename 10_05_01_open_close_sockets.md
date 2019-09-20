### Open and Close Sockets

The following functions work the same for all namespaces and communication styles.

#### Creating a Socket

```
#include  sys/socket.h

// namespace:   SOCK_STREAM, SOCK_DGRAM
// style:       PF_LOCAL, PF_INET
// protocol:    0 -> selects default protocol for specified namespace-commstyle pair.
//
// The return value from socket is the file descriptor for the new socket
int socket (int namespace, int style, int protocol)
```

#### Closing a Socket

```
// 0 -> Stop receiving data for this socket. If further data arrives, reject it. 
// 1 -> Stop trying to transmit data from this socket. Discard any data waiting to be sent. Stop looking for acknowledgement of data already sent; don't retransmit it if it is lost. 
// 2 -> Stop both reception and transmission. 
int shutdown (int socket, int how)
```

#### Socket Pairs

A socket pair is much like a pipe; the main difference is that the socket pair is bidirectional, whereas the pipe has one input-only end and one output-only end

```
// The socket pair is a full-duplex communications channel, so that both reading and writing may be performed at either end. 
// namespace -> AF_LOCAL, style -> 0
int socketpair (int namespace, int style, int protocol, int filedes[2])
```
