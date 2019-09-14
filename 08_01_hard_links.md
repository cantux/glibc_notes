### Hard Links

In POSIX systems, one file can have many names at the same time. All of the names are equally real, and no one of them is preferred to the others. 

To add a name to a file, use the link function. Creating a new link to a file does not copy the contents of the file; it simply makes a new name by which the file can be known. 

```

#include unistd.h

// Similar to link additionally it identifies its source and target using a combination of a file descriptor and a pathname. 
// if pathnames are not absolute, it is resolved relative to the corresponding file descriptor.
int linkat (int oldfd, const char *oldname, int newfd, const char *newname, int flags)
int link (const char *oldname, const char *newname)
```
