## Low Level I/O

Descriptor level functions. 


### Opening and closing files:
```
open
fclose
```

### Primitives:
```
// represents the sizes of blocks that can be read or written in a single operation.
typedef unsigned int ssize_t

// read up to size bytes
// return number of read bytes, this can be less than size
// if 0 is returned that means we are at the end of file.
// if size is 0 and 0 is returned that's not an error.
ssize_t read(int filedesc, void* buffer, size)

// similar to read but start from an offset
ssize_t pread

// 64 b wide offset
pread64


// write size number of bytes.
// return number of bytes written.
// should always be written in a for loop until returned size is 0.
// 
// once write returns, data is enqueued to be written and can be read back right away.
// but it is not necesserily written to the disk.
// - you can use fsync() to make sure data is written to the storage.
// - you can open the file in O_FSYNC mode
ssize_t write(filedesc, const void* buf, size_t size)

pwrite
pwrite64
```

### Position


