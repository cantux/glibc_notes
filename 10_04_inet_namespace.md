### The Internet Namespace

Use PF_INET and PF_INET6 symbolic names as a param to socket.

A socket address has following components:

- The address of the machine you want to connect to. 
Internet addresses can be specified in several ways:
    - Internet Address Formats, Host Addresses and Host Names discusses different specifications for addresses.

- A port number for that machine.

One must ensure that the address and port number are represented in a canonical format called *network byte order*

#### Internet Address Formats

```

#include netinet/in.h

struct sockaddr_in {
    // Address family or format of the socket address. 
    sa_family_t sin_family

    // Ipv4 address. See Host Addresses and Host Names for how to get a value to store here.
    struct in_addr sin_addr;
    
    // Port number. See Ports
    unsigned short int sin_port
}

```

#### Host Addresses

**IPv4 internet host address** is a number containing four bytes of data.

Historically host address is a divided into two parts:

- Network number
    - Local network address number within that network

In mid 90s classless addresses were introduced which changed this behavior. This document first shows class-based network, then will descrive classless addresses. 
IPV6 uses only classless addresses and therefore the following paragraphs do not apply.

##### Class-based network addresses

##### Network Number

IPv4 network numbers are registered with the Network Information Center(NIC) and are divided intro three classes, A, B and C.

0 - 127. There are only a small number of Class A networks. but they can support very large number of hosts.

Medium-sized Class B networks have two-byte network numbers, with the first byte in the range 128-191.

Class C networks are the smallest; they have three btye network numbers, with the first byte in the range 192-255. 

Thus the first 1, 2, 3 bytes of an internet address specify a network. The remaining bytes of the internet address specift the address within that network.

- class A network 0 is reserved for broadcast to all networks. In addition, the host number 0 within each network is reserved for broadcast to all hsots in that network.
These numbers are obselete now but you shouldnot use them.
- class A network 127 is reserved for loopback, you can always use 127.0.0.1 to refer the host machine.

Single machine can be a member of multiple networks, therefore it can have multiple *Internet host addresses*

How ever there is never supposed to tbe more than one machine with the same host address.

##### Classless Addresses

Class-based addresses are now ignored.

#### Host Address Data Type

Internet host addresses are represented in the smae cotexts as integers. uint32_t 

```
#include netinet/in.h

struct in_addr {
    // INADDR_LOOPBACK  : Use this constant to stand for *address of the machine* instead of finding out what it is.
    // INADDR_ANY       : See Setting Address. This is the usual address when you accept internet connections.
    // INADDR_BROADCAST : Use this to send a broadcast message.
    // INADDR_NONE      : Returned by some functions to indicate an error.
    unint_32 s_addr;    
}

struct in6_addr // 128 bit has bunch macros defined.
```

#### Byte Order
Host machines may use big or little endian addressing orders. 

Internet protocols specify a canonical byte order convention for data transmmitted over the network.

This is known as **network byte order**.

When communicating over a socket you must make sure that data in the sin_port and sin_addr members of the sockaddr_in structure are represented in network byte order.
Same goes for the data.

If you use getservbyname and gethostbyname or inet_addr to get the port number and host addresses, the values are already in network byte order
and you can copy them directly into the sockaddr_in structure.

Glibc implementation/build knows the ordering and functions are converted accordingly. This is the way to write cross platform code practice.

```
#include netinet/in.h

// Use *htons* and *nhtons* to convert values for the *sin_port* member
// Converts the uint16_t integer hostshort from host byte order to netowrk byte order.
uint16_t htons (uint16_t hostshort)
// Converts the uint16_t integer netshort from network byte order to host byte order.
uint16_t ntohs (uint16_t netshort)

// Use *htonl* to convert IPv4 addresses for the *sin_addr* member
// Converts the uint32_t integer hostlong from host byte order to network byte order
// uint32_t htonl (uint32_t hostlong)
// Converts the uint32_t integer netlong from network byte order to host byte order
// uint32_t ntohl (uint32_t netlong)
```

#### Host Address Functions

```

```
