#### Protocols Database

The communications protocol used with a socket controls low-level details of how data are exchanged. 

The network protocols that a host knows about are stored in a database. This is usually either derived from the file /etc/protocols

```
#include netdb.h

struct protoent {
    // official name of the protocol
    char *p_name

    // alternate names for the protocol, specified as an array of strings. The last element of the array is a null pointer. 
    char **p_aliases
    
    // protocol number (in host byte order); use this member as the protocol argument to socket
    int p_proto

}

// returns information about the network protocol named name
struct protoent * getprotobyname (const char *name)

// returns information about the network protocol with number protocol
struct protoent * getprotobynumber (int protocol)

// This function opens the protocols database to begin scanning it.
// If the stayopen argument is nonzero, this sets a flag so that subsequent calls to getprotobyname or getprotobynumber will not close the database (as they usually would). This makes for more efficiency if you call those functions several times, by avoiding reopening the database for each call. 
void setprotoent (int stayopen)

// returns the next entry in the protocols database. It returns a null pointer if there are no more entries. 
struct protoent * getprotoent (void)

// closes the protocols database
void endprotoent (void)
```
