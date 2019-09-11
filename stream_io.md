## I/O on Streams

For historical reasons C data structure of a stream is named FILE.

### Stream redirecting
stdin, stdout, stderr are global variable names. One can redirect those streams to files via, fclose+fopen or only freopen

### Closing
If a program terminates normally or via the exit fnc. All of the open streams are closed. How ever this is not gauranteed with abort or a fatal signal.

### Multithreading & Locking
POSIX requires stream operations to be atomic. How ever there are cases where this is not wanted or not enough. 

#### Locking for multiple operations
Imagine the case where you like to do operations in between. Here is a solution:

```
FILE *fp;
{
   ...
   flockfile (fp);
   fputs ("This is test number ", fp);
   fprintf (fp, "%d\n", test);
   funlockfile (fp)
}
```

Even better:

```
void
foo (FILE *fp)
{
  if (ftrylockfile (fp) == 0)
    {
      fputs ("in foo\n", fp);
      funlockfile (fp);          //// do not try to unlock if trylockfile returns non-zero!!!
    }
}
```

#### Unlocking
Reason why you might want to do this is to avoid overhead of locking and unlocking. Code speaks for itself:

```
void
foo (FILE *fp, char *buf)
{
  flockfile (fp);
  while (*buf != '/')
    putc_unlocked (*buf++, fp);
  funlockfile (fp);
}
```

### Types of operations

- Character Input: 
    - fgetc(FILE* stream)
    - getc: optimized version of fgetc[implemented as a macro that evaluates the stream argument more than once]
    - getc_unlocked
    - getchar(void): getc(stdin)
    - getw: reads a word. use fread instead.
- Line Input:
    - getline(char** buf, size_t* n, FILE* stream): read entire line from stream, store into buf of size n. 
        - If buf is smaller than the buf size, buf is reallocated. If buf is NULL and n is 0, buf is malloced.
        - This is a GNU extension but is the recommended way to read a line, rest of the fncs are unreliable.
    - getdelim: similar to getline but the delimeter is not a new line but the char specified.
    - fgets: if data has a NULL character you can''t tell.
    - gets: very dangerous because it provides bo protection against overflowing. 
- Unreading:
    - Streaming foobar, imagine cursor poistion is on 4th, the b. 
        - If you unread the letter "o" stream will continue as "o" and "b"
        - If you unread the letter "z" stream will continue as "z" and "b"
- Block Input/output:
    - fread
    - fwrite
    - fwrite_unlocked
    - fread_unlocked

### Text and Binary Streams

GNU systems and other POSIX compatible operating systems organize all files as uniform squences of characters.

Only reason why text streams exist is that non-GNU operating systems use different file formats and C tries to provide compatibility.

### File positioning

- ftell, ftello, ftello64: find pos
- fseek, fseeko, fseeko64(stream, offset, whence): move by offset from one of the whence constants: SEEK_SET, SEEK_CUR, SEEK_END
- rewind: set stream to the start of the stream.


