### Storage Preallocation to avoid defragmentation

Most file systems support allocating large files in a non-contiguous fashion: the file is split into fragments which are allocated sequentially, but the fragments themselves can be scattered across the disk. File systems generally try to avoid such fragmentation because it decreases performance, but if a file gradually increases in size, there might be no other option than to fragment it. In addition, many file systems support sparse files with holes: regions of null bytes for which no backing storage has been allocated by the file system. When the holes are finally overwritten with data, fragmentation can occur as well.

Explicit allocation of storage for yet-unwritten parts of the file can help the system to avoid fragmentation. Additionally, if storage pre-allocation fails, it is possible to report the out-of-disk error early, often without filling up the entire disk. However, due to deduplication, copy-on-write semantics, and file compression, such pre-allocation may not reliably prevent the out-of-disk-space error from occurring later. Checking for write errors is still required, and writes to memory-mapped regions created with mmap can still result in SIGBUS.

```
// Allocate backing store for the region of length bytes starting at byte offset in the file for the descriptor fd. The file length is increased to 'length + offset' if necessary.
// Note: If fallocate is not available (because the file system does not support it), posix_fallocate is emulated, which has the following drawbacks:
//   It is very inefficient because all file system blocks in the requested range need to be examined (even if they have been allocated before) and potentially rewritten. In contrast, with proper fallocate support (see below), the file system can examine the internal file allocation data structures and eliminate holes directly, maybe even using unwritten extents (which are pre-allocated but uninitialized on disk).
//   There is a race condition if another thread or process modifies the underlying file in the to-be-allocated area. Non-null bytes could be overwritten with null bytes.
//   If fd has been opened with the O_WRONLY flag, the function will fail with an errno value of EBADF.
//   If fd has been opened with the O_APPEND flag, the function will fail with an errno value of EBADF.
//   If length is zero, ftruncate is used to increase the file size as requested, without allocating file system blocks. There is a race condition which means that ftruncate can accidentally truncate the file if it has been extended concurrently. 
//   On Linux, if an application does not benefit from emulation or if the emulation is harmful due to its inherent race conditions, the application can use the Linux-specific fallocate function, with a zero flag argument. For the fallocate function, the GNU C Library does not perform allocation emulation if the file system does not support allocation. Instead, an EOPNOTSUPP is returned to the caller.


int posix_fallocate (int fd, off_t offset, off_t length)
int posix_fallocate64 (int fd, off64_t offset, off64_t length)
```
