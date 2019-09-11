## Stream I/O - Opening Streams

Opening a file with the fopen or fdopen creates a new stream.

```
#include stdio.h
FILE * fopen (const char *filename, const char *opentype)
```


- 'r': Open an existing file for reading only.
- 'w': Open the file for writing only. If the file already exists, it is truncated to zero length. Otherwise a new file is created.
- 'a': Open a file for append access; that is, writing at the end of file only. If the file already exists, its initial contents are unchanged and output to the stream is appended to the end of the file. Otherwise, a new, empty file is created.
- 'r+': Open an existing file for both reading and writing. The initial contents of the file are unchanged and the initial file position is at the beginning of the file.
- 'w+': Open a file for both reading and writing. If the file already exists, it is truncated to zero length. Otherwise, a new file is created.
- 'a+': Open or create file for both reading and appending. If the file exists, its initial contents are unchanged. Otherwise, a new file is created. The initial file position for reading is at the beginning of the file, but output is always appended to the end of the file. 

'+' requests a stream that can do both input and output. When using such a stream, you must call **fflush** (see Stream Buffering) or a file positioning function such as **fseek** (see File Positioning) when switching from reading to writing or vice versa. Otherwise, internal buffers might not be emptied properly. 

- 'c': The file is opened with cancellation in the I/O functions disabled.
- 'e': The underlying file descriptor will be closed if you use any of the exec... functions (see Executing a File). (This is equivalent to having set FD_CLOEXEC on that descriptor. See Descriptor Flags.)
- 'm': The file is opened and accessed using mmap. This is only supported with files opened for reading.

