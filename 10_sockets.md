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

Sockets don't imply TCP.

**Namespace** is chose for naming the socket.

A socket name("address") is meaningful only in the context of a particular namespace.

Data type to use for a spcket name may depend on the namespace.

Namespaces are also called domains.

Each namespace has a symbolic name that starts with *PF_*. 

Namespaces have address formats starting with *AF_*.

Protocol determines what low-level mechanism is used to transmit and received data.

- In order to have communication between two sockets, they must specify the same protocol.
- Each protocol is meaningful with particular style/namespace combinations and cannot be used
with inappropriate combinations. For example TCP fits only the byte stream style of communication
and the Internet namespace. 
- For each combination of style and namespace, there is a **default** protocol.
You can request for the default by specifying 0 as the protocol number.

### Comm Styles

```
// Similar to a pipe
SOCK_STREAM

SOCK_DGRAM

SOCK_RAM // low-level
```


