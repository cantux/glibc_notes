### Control Operations on Files

*fcntl* function performs the operation specified by command on the file descriptor filedes

Many of these flags are also used by the open function

```
#include fcntl.h

// F_DUPFD: Duplicate the file descriptor
#include unistd.h
// fcntl(old, F_DUPFD, 0) is equavilent to int dup(int old)
// close (new); fcntl (old, F_DUPFD, new) is equivalent to dup2(old, new)
// However, dup2 does this atomically; there is no instant in the middle of calling dup2 at which new is closed and not yet a duplicate of old. 

// F_GETFD, F_SETFD: Get. set flags associated with the file descriptor. 
// FD_CLOEXEC: file descriptor should be closed when an exec function is invoked // there are some details to this take a look when reqd.

// F_GETFL, F_SETFL : Get,set flags associated. See [File Status](07_08_file_status_flags.md) 

// F_GETLK, F_SETLK, F_SETLKW: Get/Set/SetWait or clear a file lock. See [File Locks](07_09_file_locks.md)

// F_OFD_GETLK, F_OFD_SETLK, F_OFD_SETLKW: Test/Set/SetWait an open file description lock. See Open File Description Locks. Specific to Linux.

// This function is a cancellation point, use cancellation handlers.

int fcntl (int filedes, int command, ...)
```


