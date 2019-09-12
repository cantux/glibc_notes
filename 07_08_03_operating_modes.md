### I/O Operating Modes
```
// If set, then all write operations write the data at the end of the file, extending it, regardless of the current file position. 
// This is the only reliable way to append to a file. In append mode, you are guaranteed that the data you write will always go to the current end of the file, 
// regardless of other processes writing to the file. 
// Conversely, if you simply set the file position to the end of file and write, 
// then another process can extend the file after you set the file position but before you write, 
// resulting in your data appearing someplace before the real end of file. 
int O_APPEND

//If this bit is set, read requests on the file can return immediately with a failure status if there is no input immediately available, instead of blocking. 
// Likewise, write requests can also return immediately with a failure status if the output can't be written immediately.
// Note that the O_NONBLOCK flag is overloaded as both an I/O operating mode and a file name translation flag; see Open-time Flags. 
int O_NONBLOCK

// The remaining operating modes are BSD and GNU extensions. They exist only on some systems. On other systems, these macros are not defined.

// If set, then SIGIO signals will be generated when input is available. See Interrupt Input.
int O_ASYNC

// The bit that enables synchronous writing for the file. If set, each write call will make sure the data is reliably stored on disk before returning.
int O_FSYNC
int O_SYNC

