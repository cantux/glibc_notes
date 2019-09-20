## Sockets

Socket is a generalized interprocess communication channel.

Like a pipe, socket is represented as a file descriptor.

Unlike pipes, 

- sockets support communication between unrelated processes
- even processes running on different machines
- telnet

```
#include sys/socket.h
```

When you create a a socket you must specify:

- Style communication you want to use
- type of protocol that should implement it.

### Communication Style

**Communication Style** of a socket defines the user-level semantics of sending and receiving data on the socket.

Choosing a **Communication Style** specifies the answers to questions such as these:

- What are the units of data transmissions? Some regard the data as a seq of bytes with no larger structure; others group the bytes into records(packets).
- Can data be lost during normal operation? Some gaurantee all data arrives in order some don't
- Is communication entirely with one partner? Some like telephone call some like sending an email, specify addresses.

Sockets don't imply TCP.

```
// Similar to a pipe
SOCK_STREAM

SOCK_DGRAM // See Datagrams

SOCK_RAM // low-level
```

### Namespace

**Namespace** is chose for naming the socket.

A socket name("address") is meaningful only in the context of a particular namespace.

Data type to use for a socket name may depend on the namespace.

Namespaces are also called domains.

Each namespace has a symbolic name that starts with *PF_*. 

Namespaces have address formats starting with *AF_*.

### Protocol

Protocol determines what low-level mechanism is used to transmit and received data.

- In order to have communication between two sockets, they must specify the same protocol.
- Each protocol is meaningful with particular style/namespace combinations and cannot be used
with inappropriate combinations. For example TCP fits only the byte stream style of communication
and the Internet namespace. 
- For each combination of style and namespace, there is a **default** protocol.
You can request for the default by specifying 0 as the protocol number.



