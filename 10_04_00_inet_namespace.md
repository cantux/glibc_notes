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


