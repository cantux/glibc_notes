### Waiting for I/O

Sometimes a program needs to accept input on multiple input channels whenever input arrives.

One example is a device with multiple inputs that is connected via async serial interfaces.
Other is a program that acts as a server to several other processes via pipes or sockets. 

You cannot normally use read for this purpose, because this blocks the program until input is available on one particular file descriptor; input on other channels won't wake it up.

You could set nonblocking mode and poll each file descriptor in turn, but this is very inefficient. 

A better solution is to use the *select* function. This blocks the program until

- input or output is ready on a specified set of file descriptors, 
- until a timer expires, 

whichever comes first.

```
#include sys/types.h

// fd_set: a bit array
// Following are the macros to manipulate it

// The value of this macro is the maximum number of file descriptors that a fd_set object can hold information about.
// There is no limit to the number of descriptors open min most of the systems.
int FD_SETSIZE

// This macro initializes the file descriptor set set to be the empty set.
void FD_ZERO (fd_set *set)

// This macro adds filedes to the file descriptor set set. 
void FD_SET (int filedes, fd_set *set)

void FD_CLR (int filedes, fd_set *set)
int FD_ISSET (int filedes, const fd_set *set)


// The select function blocks the calling process until there is activity on any of the specified sets of file descriptors, or until the timeout period has expired.
// The file descriptors specified by the read-fds, the write-fds and the except-fds file descriptors are checked for read, write and exceptional conditions respectively. Null to ignore any of them. 

// A server socket is considered ready for reading if there is a pending connection which can be accepted with accept.
    // The server's original socket does not become part of the connection; instead, accept makes a new socket which participates in the connection. accept returns the descriptor for this socket.
    // O_NONBLOCK file operating mode.

// The select function checks only the first nfds file descriptors. The usual thing is to pass FD_SETSIZE as the value of this argument
int select (int nfds, fd_set *read-fds, fd_set *write-fds, fd_set *except-fds, struct timeval *timeout)

```

#### Examples
```
include <errno.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/time.h>


int
input_timeout (int filedes, unsigned int seconds)
{
  fd_set set;
  struct timeval timeout;


  /* Initialize the file descriptor set. */
  FD_ZERO (&set);
  FD_SET (filedes, &set);

  /* Initialize the timeout data structure. */
  timeout.tv_sec = seconds;
  timeout.tv_usec = 0;

  /* select returns 0 if timeout, 1 if input available, -1 if error. */
  return TEMP_FAILURE_RETRY (select (FD_SETSIZE,
                                     &set, NULL, NULL,
                                     &timeout));
}


int
main (void)
{
  fprintf (stderr, "select returned %d.\n",
           input_timeout (STDIN_FILENO, 5));
  return 0;
}

```

```

#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

#define PORT    5555
#define MAXMSG  512

int
read_from_client (int filedes)
{
  char buffer[MAXMSG];
  int nbytes;

  nbytes = read (filedes, buffer, MAXMSG);
  if (nbytes < 0)
    {
      /* Read error. */
      perror ("read");
      exit (EXIT_FAILURE);
    }
  else if (nbytes == 0)
    /* End-of-file. */
    return -1;
  else
    {
      /* Data read. */
      fprintf (stderr, "Server: got message: `%s'\n", buffer);
      return 0;
    }
}

int
main (void)
{
  extern int make_socket (uint16_t port);
  int sock;
  fd_set active_fd_set, read_fd_set;
  int i;
  struct sockaddr_in clientname;
  size_t size;

  /* Create the socket and set it up to accept connections. */
  sock = make_socket (PORT);
  if (listen (sock, 1) < 0)
    {
      perror ("listen");
      exit (EXIT_FAILURE);
    }

  /* Initialize the set of active sockets. */
  FD_ZERO (&active_fd_set);
  FD_SET (sock, &active_fd_set);

  while (1)
    {
      /* Block until input arrives on one or more active sockets. */
      read_fd_set = active_fd_set;
      if (select (FD_SETSIZE, &read_fd_set, NULL, NULL, NULL) < 0)
        {
          perror ("select");
          exit (EXIT_FAILURE);
        }

      /* Service all the sockets with input pending. */
      for (i = 0; i < FD_SETSIZE; ++i)
        if (FD_ISSET (i, &read_fd_set))
          {
            if (i == sock)
              {
                /* Connection request on original socket. */
                int new;
                size = sizeof (clientname);
                new = accept (sock,
                              (struct sockaddr *) &clientname,
                              &size);
                if (new < 0)
                  {
                    perror ("accept");
                    exit (EXIT_FAILURE);
                  }
                fprintf (stderr,
                         "Server: connect from host %s, port %hd.\n",
                         inet_ntoa (clientname.sin_addr),
                         ntohs (clientname.sin_port));
                FD_SET (new, &active_fd_set);
              }
            else
              {
                /* Data arriving on an already-connected socket. */
                if (read_from_client (i) < 0)
                  {
                    close (i);
                    FD_CLR (i, &active_fd_set);
                  }
              }
          }
    }
}
```

