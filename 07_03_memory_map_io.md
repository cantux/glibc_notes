### Memory-mapped I/O

mmap is more efficient than read or write. Swapping is automatically handled.

Memory mapping only works on entire pages of memory. Thus addresses for mapping must be page-aligned.

Use following To determine the default size of a page:
```
size_t page_size = (size_t) sysconf(_SC_PAGESIZE);
```

#### MMAP function

Bow to the gods.

```
#include sys/mman.h

void * mmap (void *address, size_t length, int protect, int flags, int filedes, off_t offset)


```

New threads and subprocesses inherit the access rights of the current thread. 
If a protection key is allocated subsequently, existing threads (except the current) will use an unspecified system default for the access rights associated with newly allocated keys. 

To allocate blocks much larger than a page, use mmap(anonymous or via /dev/zero).

#### Memory Protection

Keys:

- PROT_WRITE -> The memory can be written to.
- PROT_READ -> The memory can be read. On some architectures, this flag implies that the memory can be executed as well (as if PROT_EXEC had been specified at the same time).
- PROT_EXEC -> The memory can be used to store instructions which can then be executed. implies PROT_READ
- PROT_NONE -> This flag must be specified on its own.
The memory is reserved, but cannot be read, written, or executed. 
If this flag is specified in a call to mmap, a virtual memory area will be set aside for future use in the process, 
and mmap calls without the MAP_FIXED flag will not use it for subsequent allocations. 
For anonymous mappings, the kernel will not reserve any physical memory for the allocation at the time the mapping is created. 

#### Flags

- *MAP_PRIVATE*: This specifies that writes to the region should never be written back to the attached file. 

Instead, a copy is made for the process, and the region will be swapped normally if memory runs low. No other process will see the changes.

Since private mappings effectively revert to ordinary memory when written to, you must have enough virtual memory for a copy of the entire mmapped region if you use this mode with PROT_WRITE.

- *MAP_SHARED*: This specifies that writes to the region will be written back to the file. Changes made will be shared immediately with other processes mmaping the same file.

Note that actual writing may take place at any time. You need to use msync, described below, if it is important that other processes using conventional I/O get a consistent view of the file.

- *MAP_FIXED*: This forces the system to use the exact mapping address specified in address and fail if it can't.

- *MAP_ANON*: This flag tells the system to create an anonymous mapping, not connected to a file. filedes and offset are ignored, and the region is initialized with zeros.

Anonymous maps are used as the basic primitive to extend the heap on some systems. They are also useful to share data between multiple tasks without creating a file. 

### msync, madvise

```
// When using shared mappings, the kernel can write the file at any time before the mapping is removed. 
// To be certain data has actually been written to the file and will be accessible to non-memory-mapped I/O, it is necessary to use this function. 
// MS_SYNC -> This flag makes sure the data is actually written to disk. Normally msync only makes sure that accesses to a file with conventional I/O reflect the recent changes.
// MS_ASYNC -> This tells msync to begin the synchronization, but not to wait for it to complete.
int msync (void *address, size_t length, int flags)

// MADV_NORMAL The region should receive no further special treatment.
// MADV_RANDOM The region will be accessed via random page references. The kernel should page-in the minimal number of pages for each page fault.
// MADV_SEQUENTIAL The region will be accessed via sequential page references. This may cause the kernel to aggressively read-ahead, expecting further sequential references after any page fault within this region.
// MADV_WILLNEED The region will be needed. The pages within this region may be pre-faulted in by the kernel.
// MADV_DONTNEED The region is no longer needed. The kernel may free these pages, causing any changes to the pages to be lost, as well as swapped out pages to be discarded.
// MADV_HUGEPAGE Indicate that it is beneficial to increase the page size for this mapping. This can improve performance for larger mappings because the system needs to handle far fewer pages. However, if parts of the mapping are frequently transferred between storage or different nodes, performance may suffer because individual transfers can become substantially larger due to the increased page size. This flag is specific to Linux.
// MADV_NOHUGEPAGE Undo the effect of a previous MADV_HUGEPAGE advice. This flag is specific to Linux.
// used to provide the system with advice about the intended usage patterns of the memory region starting at addr and extending length bytes
int madvise (void *addr, size_t length, int advice)

```

### shm_open, shm_unlink, memfd_create

```
// This function returns a file descriptor that can be used to allocate shared memory via mmap. Unrelated processes can use same name to create or open existing shared memory objects. 
int shm_open (const char *name, int oflag, mode_t mode)

// This function is the inverse of shm_open and removes the object with the given name previously created by shm_open. 
int shm_unlink (const char *name)

// name argument is used for debugging purposes only (e.g., will appear in /proc) 
// separate invocations of memfd_create with the same name will not return descriptors for the same region of memory
int memfd_create (const char *name, unsigned int flags)
```




