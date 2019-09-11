

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
    - Symbolic vs hard: symbolic can be created across filesystems.
- FIFO: Named Pipe
- Socket

#### Interpreter Files:

Text files with #!. Recognized by kernel. 
- Kernel starts interpreter specified by the pathname. 
- Redirects rest of the file to interpreter's stdin.
 
### File Descriptors

- Small integers the kernel uses to identify open files in a process.
- Index in a table maintained by kernel.




