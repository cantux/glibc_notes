### Open File Description Locks

*Open File Description*: an instance of an open file, usually created by a call to open

*Open File Descriptor*: a numeric value that refers to the open file description

As with process-associated locks, open file description locks are advisory. 

Open file description record locks are associated with an open file description rather than a process

Using fcntl to apply an open file description lock on a region that already has an existing open file description lock that was created via the same file descriptor will never cause a lock conflict. 

Open file description locks are also inherited by child processes across fork, or clone with CLONE_FILES set (see Creating a Process), along with the file descriptor. 

??!!Using **dup** to copy a file descriptor does not give you a new open file description!! but rather copies a reference to an existing file description and assigns it to a new file descriptor.??
??Thus, open file description locks set on a file descriptor cloned by dup will never conflict with open file description locks set on the original descriptor since they refer to the same open file description. Depending on the range and type of lock involved, the original lock may be modified by a F_OFD_SETLK or F_OFD_SETLKW command in this situation however.??

Open file description locks always conflict with process-associated locks, even if acquired by the same process or on the same open file descriptor. 

```
#include fcntl.h

struct flock {
    // F_RDLCK, F_WRLCK, or F_UNLCK
    short int l_type;

    // SEEK_SET, SEEK_CUR, or SEEK_END. 
    short int l_whence;

    off_t l_start
    off_t l_len
    pid_t l_pid
}

fcntl (filedes, F_OFD_GETLK, lockp)
fcntl (filedes, F_OFD_SETLK, lockp)
int F_OFD_SETLKW
```

Program below will create 3 threads that loops 5 times.

```
#define _GNU_SOURCE
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#include <pthread.h>

#define FILENAME        "/tmp/foo"
#define NUM_THREADS     3
#define ITERATIONS      5

void *
thread_start (void *arg)
{
  int i, fd, len;
  long tid = (long) arg;
  char buf[256];
  struct flock lck = {
    .l_whence = SEEK_SET,
    .l_start = 0,
    .l_len = 1,
  };

  fd = open ("/tmp/foo", O_RDWR | O_CREAT, 0666);

  for (i = 0; i < ITERATIONS; i++)
    {
      lck.l_type = F_WRLCK;
      fcntl (fd, F_OFD_SETLKW, &lck);

      len = sprintf (buf, "%d: tid=%ld fd=%d\n", i, tid, fd);

      lseek (fd, 0, SEEK_END);
      write (fd, buf, len);
      fsync (fd);

      lck.l_type = F_UNLCK;
      fcntl (fd, F_OFD_SETLK, &lck);

      /* sleep to ensure lock is yielded to another thread */
      usleep (1);
    }
  pthread_exit (NULL);
}

int
main (int argc, char **argv)
{
  long i;
  pthread_t threads[NUM_THREADS];

  truncate (FILENAME, 0);

  for (i = 0; i < NUM_THREADS; i++)
    pthread_create (&threads[i], NULL, thread_start, (void *) i);

  pthread_exit (NULL);
  return 0;
}
```
