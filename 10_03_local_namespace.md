### The Local Namespace

Namespace with the symbolic name **PF_LOCAL**. 

The Local Namespace is also knwown as *Unix domain sockets*. 

Another name is *file* namespace.

#### Concepts

The local namespace socket addresses are file names. It is common to put these files in the /tmp directory.

Local namespace supports just one protocol for any comm style(DGRAM or STREAM), it is protocol number 0.

One cannot connect to this socket from a remote machine. 
One can list the sockets and local namespace sockets will appear in that list but connecting to it never succeds.
Some programs take advantage of this(by trying to connect, find out about the failure, associate this with the process ID) but it is not recommended to use this
information as this failure spec can change.

File must be deleted by unlink or remove.


```
#include sys/socket.h

//PF_LOCAL, PF_UNIX, PF_FILE are used as argument to socket or socketpair

struct sockaddr_un {
    // Identifies address family or format of the socket address. AF_LOCAL for local namespace
    sort int sun_family;
    
    // File name to use.
    char sun_path[108];
}

//Computes the length parameter for a socket address in the local namespace as the sum of the size of the sun_familty component and the string length.

int SUN_LEN(struct sockaddr_un* ptr)
```

Example

```
#include <stddef.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/un.h>

int
make_named_socket (const char *filename)
{
  struct sockaddr_un name;
  int sock;
  size_t size;

  /* Create the socket. */
  sock = socket (PF_LOCAL, SOCK_DGRAM, 0);
  if (sock < 0)
    {
      perror ("socket");
      exit (EXIT_FAILURE);
    }

  /* Bind a name to the socket. */
  name.sun_family = AF_LOCAL;
  strncpy (name.sun_path, filename, sizeof (name.sun_path));
  name.sun_path[sizeof (name.sun_path) - 1] = '\0';

  /* The size of the address is
     the offset of the start of the filename,
     plus its length (not including the terminating null byte).
     Alternatively you can just do:
     size = SUN_LEN (&name);
 */
  size = (offsetof (struct sockaddr_un, sun_path)
          + strlen (name.sun_path));

  if (bind (sock, (struct sockaddr *) &name, size) < 0)
    {
      perror ("bind");
      exit (EXIT_FAILURE);
    }

  return sock;
}
```



