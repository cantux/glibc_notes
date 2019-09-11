
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


