### Low Level I/O - Scatter Gather
```
#include uio.h

struct iovec {
    void* iov_base; // base address of the buffer
    size_t iov_len; // length of the buffer
}

// Reads data from filedes and scatters it into the buffers given in vector of size count.
// As each buffer is filled, data is sent to the next
ssize_t readv (int filedes, const struct iovec *vector, int count)

// DAFUG??
ssize_t writev (int filedes, const struct iovec *vector, int count)
```
