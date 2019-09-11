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

```
// whence -> 
// SEEK_CUR: specifies offset to be interpreted as count of characters from the current position
// SEEK_SET: specifies offset to be interpreted as count of characters from the start
// SEEK_END: positive is interesting. if you set position past current end and write data you will extend the file with zeros up to that position.
//              Setting position past end does not make the file longer but subsequent write at the end will extend the file. 
//              Extending the file like this will create a "hole"
//
//              If you append to the file, setting the file position to the current end of file with SEEK_END is not sufficient.
//              Another ptocess may write more data after you seek but before you write.
//              Extending the fuke so the position you write onto clobbers their data. 
//              Instead, use the O_APPEND operating mode.
//
// return current position measured in bytes, -1 on error
//
// This function is a cancellation point. its user thread should have cancellation handlers.
lseek(filedesc, offset, whence)
lseek64
```

#### dup
Descriptors that come from separate calls to open have independent file positions; using lseek on one descriptor has no effect on the other.

```
{
  int d1, d2;
  char buf[4];
  d1 = open ("foo", O_RDONLY);
  d2 = open ("foo", O_RDONLY);
  lseek (d1, 1024, SEEK_SET);
  read (d2, buf, 4);
}
```
will read the first four characters of the file foo

```
{
  int d1, d2, d3;
  char buf1[4], buf2[4];
  d1 = open ("foo", O_RDONLY);
  d2 = dup (d1);
  d3 = dup (d2);
  lseek (d3, 1024, SEEK_SET);
  read (d1, buf1, 4);
  read (d2, buf2, 4);
}
```

 will read four characters starting with the 1024'th character of foo
