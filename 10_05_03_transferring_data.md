#### Transferring Data

Once a socket has been connected to a peer, you can use the ordinary **read** and **write** opertaions to transfer data.

Socket is a two way communication channel so read and write can be performed at either end.

You must use **recv** and **send** functions to use specific I/O modes. See Data Options which are used as special flags to these functions.

```
#include "sys/socket.h"

// if you use 0 as a flag it's the smae as write or read.
ssize_t send (int socket, const void *buffer, size_t size, int flags)

// If nonblocking mode is set for socket, and no data are available to be read, recv fails immediately rather than waiting
ssize_t recv (int socket, void *buffer, size_t size, int flags)
```

##### Socket Data Options

```
// delivered with higher priority than ordinary data.  Typically the reason for sending out-of-band data is to send notice of an exceptional condition.
// use recv with the MSG_OOB flag
//
// When a socket finds that out-of-band data are on their way, it sends a SIGURG signal to the owner process or process group of the socket. 
// You can specify the owner using the F_SETOWN command to the fcntl function; see Interrupt Input. You must also establish a handler for this signal, in order to take appropriate action such as reading the out-of-band data.
// Alternatively, you can test for pending out-of-band data, or wait until there is out-of-band data, using the select function; it can wait for an exceptional condition on the socket.
// Notification of out-of-band data (whether with SIGURG or with select) indicates that out-of-band data are on the way; the data may not actually arrive until later. If you try to read the out-of-band data before it arrives, recv fails with an EWOULDBLOCK error. 
int MSG_OOB

/// Look at the data but don't remove it from the input queue. This is only meaningful with input functions such as recv, not with send. 
int MSG_PEEK

// Don't include routing information in the message. This is only meaningful with output operations, and is usually only of interest for diagnostic or routing programs. We don't try to explain it here. 
int MSG_DONTROUTE
```

###### Out-of-Band Data Cont.

Sending out-of-band data automatically places a "mark" in the stream of ordinary data, showing where in the sequence the out-of-band data "would have been". This is useful when the meaning of out-of-band data is "cancel everything sent so far". Here is how you can test, in the receiving process, whether any ordinary data was sent before the mark:

```
success = ioctl (socket, SIOCATMARK, &atmark);
```

The integer variable atmark is set to a nonzero value if the socket's read pointer has reached the "mark". 

Discarding any ordinary data preceding the out-of-band mark: 

```
int
discard_until_mark (int socket)
{
  while (1)
    {
      /* This is not an arbitrary limit; any size will do.  */
      char buffer[1024];
      int atmark, success;

      /* If we have reached the mark, return.  */
      success = ioctl (socket, SIOCATMARK, &atmark);
      if (success < 0)
        perror ("ioctl");
      if (result)
        return;

      /* Otherwise, read a bunch of ordinary data and discard it.
         This is guaranteed not to read past the mark
         if it starts before the mark.  */
      success = read (socket, buffer, sizeof buffer);
      if (success < 0)
        perror ("read");
    }
}
```

If you don't want to discard the ordinary data preceding the mark, you may need to read some of it anyway, to make room in internal system buffers for the out-of-band data. 
If you try to read out-of-band data and get an EWOULDBLOCK error, try reading some ordinary data (saving it so that you can use it when you want it) and see if that makes room. Here is an example: 

```
struct buffer
{
  char *buf;
  int size;
  struct buffer *next;
};

/* Read the out-of-band data from SOCKET and return it
   as a 'struct buffer', which records the address of the data
   and its size.

   It may be necessary to read some ordinary data
   in order to make room for the out-of-band data.
   If so, the ordinary data are saved as a chain of buffers
   found in the 'next' field of the value.  */

struct buffer *
read_oob (int socket)
{
  struct buffer *tail = 0;
  struct buffer *list = 0;

  while (1)
    {
      /* This is an arbitrary limit.
         Does anyone know how to do this without a limit?  */
#define BUF_SZ 1024
      char *buf = (char *) xmalloc (BUF_SZ);
      int success;
      int atmark;

      /* Try again to read the out-of-band data.  */
      success = recv (socket, buf, BUF_SZ, MSG_OOB);
      if (success >= 0)
        {
          /* We got it, so return it.  */
          struct buffer *link
            = (struct buffer *) xmalloc (sizeof (struct buffer));
          link->buf = buf;
          link->size = success;
          link->next = list;
          return link;
        }

      /* If we fail, see if we are at the mark.  */
      success = ioctl (socket, SIOCATMARK, &atmark);
      if (success < 0)
        perror ("ioctl");
      if (atmark)
        {
          /* At the mark; skipping past more ordinary data cannot help.
             So just wait a while.  */
          sleep (1);
          continue;
        }

      /* Otherwise, read a bunch of ordinary data and save it.
         This is guaranteed not to read past the mark
         if it starts before the mark.  */
      success = read (socket, buf, BUF_SZ);
      if (success < 0)
        perror ("read");

      /* Save this data in the buffer list.  */
      {
        struct buffer *link
          = (struct buffer *) xmalloc (sizeof (struct buffer));
        link->buf = buf;
        link->size = success;

        /* Add the new link to the end of the list.  */
        if (tail)
          tail->next = link;
        else
          list = link;
        tail = link;
      }
    }
}
```
