### File Locks

This section describes record locks that are associated with the process. 

There is also a different type of record lock that is associated with the open file description instead of the process. See Open File Description Locks. 

An **exclusive** or **write** lock gives a process exclusive access for writing to the specified part of the file. No other process can lock that part.

A **shared** or **read** lock prohibits any other process from requesting a write lock on the specified part of the file. How ever other processes can request locks.

read and write functions do not actually check whether there are any locks in place. If you want to implement a locking protocol for a file shared by multipleprocesses, 
your application must do explicit fcntl calls to requst and clear locks at the appropriate points.

Locks are associated with processes. A process can only have one kind of lock set for each byte of a given file.

When any descriptor for that file is closed by the process all of the locks that process golds on that file are released,
even if the locks were made using the descriptors that remian open. (How about linked shared channel difference??)

Locks are released when process exits.

Locks are not inherited by child processes create using fork.

!! Remember that file locks are only an advisory protocol for controlling access to a file. There is still potential for access to the file by programs that don't use the lock protocol.

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

fcntl (filedes, F_GETLK, lockp)


fcntl (filedes, F_SETLK, lockp)
// is this buffered why is there a wait?? is all the fcntl buffered?? how about open-time operations??
fcntl (filedes, F_SETLKW, lockp)  
```


