### Interrupt-driven Input

If you set the O_ASYNC status flag on a file descriptor(see File Status Flags), 
a SIGIO signal is sent whenever input or output becomes possible on that file descriptor.

??The process or process group to receive the signal can be selected by using the F_SETOWN command to the fcntl function.??

```
#include fcntl.h

// specify that it should get information about the process or process group to which SIGIO signals are sent
int F_GETOWN

// to specify that it should set the process or process group to which SIGIO signals are sent
fcntl (filedes, F_SETOWN, pid)
```



