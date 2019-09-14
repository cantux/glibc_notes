### Symbolic Links

This is a kind of "file" that is essentially a pointer to another file name. 

Unlike hard links, symbolic links can be made to directories or across file systems with no restrictions. 

You can also make a symbolic link to a name which is not the name of any file.


```
#include sys/param.h
// specifies how many symlinks some function will follow 
int MAXSYMLINKS


#include unistd.h

// a symbolic link to oldname named newname. 
int symlink (const char *oldname, const char *newname)

// gets the value of the symbolic link filename
// The file name that the link points to is copied into buffer !!!! This file name string is not null-terminated
// See example below for fair use.
ssize_t readlink (const char *filename, char *buffer, size_t size)

// A call to realpath where the resolved parameter is NULL behaves exactly like canonicalize_file_name. 
// returns the absolute name of the file named by name which contains no ., .. components nor any repeated path separators (/) or symlinks.
char * canonicalize_file_name (const char *name)
char * realpath (const char *restrict name, char *restrict resolved)
```

Followin gist is a good way to use readlink

```
char *
readlink_malloc (const char *filename)
{
  int size = 100;
  char *buffer = NULL;

  while (1)
    {
      buffer = (char *) xrealloc (buffer, size);
      int nchars = readlink (filename, buffer, size);
      if (nchars < 0)
        {
          free (buffer);
          return NULL;
        }
      if (nchars < size)
        return buffer;
      size *= 2;
    }
}
```
