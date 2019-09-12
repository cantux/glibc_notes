### Open-time Flags

```
// O_CREAT, O_EXCL
// O_NOLINK: If the named file is a symbolic link, open the link itself instead of the file it refers to.

// O_NONBLOCK: This prevents open from blocking for a "long time" to open the file. 
// This is only meaningful for some kinds of files, usually devices such as serial ports; when it is not meaningful, it is harmless and ignored. 
// Often, opening a port to a modem blocks until the modem reports carrier detection; if O_NONBLOCK is specified, open will return immediately without a carrier.
// Note that the O_NONBLOCK flag is overloaded as both an I/O operating mode and a file name translation flag. 
// This means that specifying O_NONBLOCK in open also sets nonblocking I/O mode; see Operating Modes. 
// To open the file without blocking but do normal I/O that blocks, you must call open with O_NONBLOCK set and then call fcntl to turn the bit off. 

// int O_SHLOCK: Acquire a shared lock on the file, as with flock. See File Locks.
// If O_CREAT is specified, the locking is done atomically when creating the file. You are guaranteed that no other process will get the lock on the new file first. 

// int O_EXLOCK
// Acquire an exclusive lock on the file, as with flock. See File Locks. This is atomic like O_SHLOCK. 
