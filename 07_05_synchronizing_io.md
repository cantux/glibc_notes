## Synchronizing I/O

Even if a write system call returns, this does not mean the data is actually written to the media, e.g., the disk. 

```
#include unistd.h
// A call to this function will not return as long as there is data which has not been written to the device. 
// All dirty buffers in the kernel will be written and so an overall consistent system can be achieved (if no other process in parallel writes data). 
void sync (void)

// The fsync function can be used to make sure all data associated with the open file fildes is written to the device associated with the descriptor. 
// The function call does not return unless all actions have finished. 
int fsync (int fildes)
```
