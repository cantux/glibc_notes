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
#include arpa/inet.h
// converts the IPv4 Internet host address name from the standard numbers-and-dots notation into binary data and stores it in the struct in_addr that addr points to. inet_aton returns nonzero if the address is valid, zero if not. 
int inet_aton (const char *name, struct in_addr *addr)
// use inet aton instead of this one.
uint32_t inet_addr (const char *name)

// converts the IPv4 Internet host address addr to a string in the standard numbers-and-dots notation. The return value is a pointer into a statically-allocated buffer. Subsequent calls will overwrite the same buffer, so you should copy the string if you need to save it. 
char * inet_ntoa (struct in_addr addr)

// makes an IPv4 Internet host address by combining the network number net with the local-address-within-network number local. 
struct in_addr inet_makeaddr (uint32_t net, uint32_t local)

uint32_t inet_lnaof (struct in_addr addr)// OBSELETE returns the local-address-within-network part of the Internet host address addr. // does not work with classless addresses.
inet_netof (struct in_addr addr)// OBSELETE returns the network number part of the Internet host address addr. 

int inet_pton (int af, const char *cp, void *buf) // convert presentation (textual) to network (binary) format
const char * inet_ntop (int af, const void *cp, char *buf, socklen_t len) // convert network (binary) to presentation (textual) form
```

#### Host Names

Besides the octal with dots host can have a symbolic name as well.

Internally system uses a database to keep track of the mapping between a hostnames and host numbers.

Internally that db is at /etc/hosts or outside residing as a name server. Functions for accessing this db are following

```
#include "netdb.h"

struct hostent {
    // official name of the host
    char* h_name;
    // Alternative names for the gost, null terminated vector of strings
    char** h_aliases;
    // Host address type, AF_INET or AF_INET6
    int h_addrtype;
    // Length, in bytes, of each address.
    int h_length;
    // Vector of addresses for the gost. Terminated by Null ptr.
    // Recall that the host might be connected to muliple networks and have different addresses on each one.
    char ** h_addr_list;
    char* h_addr; // h_addr_list[0]
}

// returns information about the host named name
struct hostent * gethostbyname (const char *name)
struct hostent * gethostbyname2 (const char *name, int af)

// returns information about the host with Internet address addr
struct hostent * gethostbyaddr (const void *addr, socklen_t length, int format)

// The lookup functions above all have one thing in common: they are not reentrant and therefore unusable in multi-threaded applications
// Therefore provides the GNU C Library a new set of functions which can be used in this context. 


// returns information about the host named **name**
int gethostbyname_r (const char *restrict name, struct hostent *restrict result_buf, char *restrict buf, size_t buflen, struct hostent **restrict result, int *restrict h_errnop)
// returns information about the host with Internet address **addr**
int gethostbyaddr_r (const void *addr, socklen_t length, int format, struct hostent *restrict result_buf, char *restrict buf, size_t buflen, struct hostent **restrict result, int *restrict h_errnop)

struct hostent *
gethostname (char *host)
{
  struct hostent *hostbuf, *hp;
  size_t hstbuflen;
  char *tmphstbuf;
  int res;
  int herr;

  hostbuf = malloc (sizeof (struct hostent));
  hstbuflen = 1024;
  tmphstbuf = malloc (hstbuflen);

  while ((res = gethostbyname_r (host, hostbuf, tmphstbuf, hstbuflen,
                                 &hp, &herr)) == ERANGE)
    {
      /* Enlarge the buffer.  */
      hstbuflen *= 2;
      tmphstbuf = realloc (tmphstbuf, hstbuflen);
    }

  free (tmphstbuf);
  /*  Check for errors.  */
  if (res || hp == NULL)
    return NULL;
  return hp;
}

// scan the entire hosts database one entry at a time using sethostent, gethostent and endhostent. 
// Be careful when using these functions because they are not reentrant. 

//  opens the hosts database to begin scanning it
void sethostent (int stayopen)
// returns the next entry in the hosts database. It returns a null pointer if there are no more entries. 
struct hostent * gethostent (void)
void endhostent (void) // closes the hosts database. 
```


