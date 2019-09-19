
#### Services Database

The database keeps track of well-known services. It is usually either the file /etc/services or an equivalent from a name server.

```
#include netdb.h

struct servent { 
    // "official" name of the service. 
    char *s_name
    
    // alternate names for the service, represented as an array of strings. A null pointer terminates the array. 
    char **s_aliases

    // port number for the service
    int s_port

    //  name of the protocol to use with this service. See Protocol Database
    char *s_proto
}

// Returns information about the service named name using protocol proto.
// This function is useful for servers as well as for clients; servers use it to determine which port they should listen on (see Listening). 
struct servent * getservbyname (const char *name, const char *proto)

// returns information about the service at port port using protocol proto
struct servent * getservbyport (int port, const char *proto)

// opens the services database to begin scanning it
void setservent (int stayopen)
// returns the next entry in the services database
struct servent * getservent (void)
void endservent (void) // closes the services database. 
```


