# Unix System Interface

Notes and gists from the Unix System Interface - C Programming Lang. 2nd Ed.

## Some history

Starts with Multics, a collaboration project between multiple groups. Had all the good ideas but eventually blobbed and failed.

AT&T Bell Labs still liked the idea and they designed a new filesystem. Then added an assembler, shell, process management and basic I/O --> UNIX!

C was developed to facilitate UNIX. C's and UNIX' success seems like a chicken and egg problem to me. Together they created an ecosystem. 

Hardware developers adopted it. Standards emerged, ANSI C, IEEE POSIX, endless list of communications protocols supported by low-level I/O interfaces.

AND of course the final nail in the coffin: the positive-feedback loop(applies to any ecosystem) of dependent/convinient software.

Imagine the slimey evolution. Life-like emergence of the modern computer similar to self replicating crystals to DNA.

## File Types
Innovation: PIPES(orgasmic noises) Though it is more organic then it sounds-as with everything-. 

Hmm, now how could one come up with this sort of a facility? Given that programs should have low coupling. "Take over where I finish, oh and we like linebreaks" --> Developers just needed their programs to be able to communicate with each other.

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

## Access BS

I can't possibly be bothered by this. Just don't superuser your way around. You will get a feel for this. 

Remember the access modes(ordered): owner, group, world

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
 

