

## File Types
- Regular Files: data
- Directory Files: utility crap
- Character special: keyboard, mouse
- Block special: disk, tape, mountable
- Device Files: some arbitarary handle implements I/O
    - Enumerate devices through universal interfaces. 
        - USB, PCIe, if using a stupid interface, I'd imagine you can load a module that will be automatically invoked at some point to look into it.
    - Device driver should be loaded as a kernel module and an identifier should be found in some map(major/minor dev numbers).
    - Driver implements communications by the means of I/O. (File I/O, Named Pipes?, SysV Shared Mem?, Signals?, What does POSIX say about this?)
- Links, hard, symbolic
- FIFO: Named Pipe
- Socket


## I/O

### File Descriptors

- Small integers the kernel uses to identify open files in a process.
- Index in a table maintained by kernel.

### Std I/O vs Unbuffered I/O

- These properties regarding the files are dependent on the settings of the stream and/or its underlying desctiptor.
- These properties are not dependent on the used function.
- **setvbuf** control buffering of C Runtime Library. options are buffered, linebuffered or unbuffered.
- All std except error is buffered. Since error is hopefully rare and critical it is unbuffered.

#### Std I/O:

- Default file descriptors for scanf, printf
- Can be redirected
- Tries to reduce syscalls and disk I/O. Instead of going over byte by byte read/write bunch of blocks at a time.

#### Unbuffered I/O:

- Standard of the File I/O.
- Default for open, read, write, lseek, close.

#### Different Levels of Buffering:

```
+-------------------+-------------------+
| Process A         | Process B         |
+-------------------+-------------------+
| C runtime library | C runtime library | C RTL buffers
+-------------------+-------------------+
|               OS caches               | Operating system buffers
+---------------------------------------+
|      Disk controller hardware cache   | Disk hardware buffers
+---------------------------------------+
|                   Disk                |
+---------------------------------------+
```

#### Interpreter Files:

Text files with #!. Recognized by kernel. 
- Kernel starts interpreter specified by the pathname. 
- Redirects rest of the file to interpreter's stdin.
 

