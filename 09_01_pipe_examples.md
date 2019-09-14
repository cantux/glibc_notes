### Pipes FIFOs Examples

- Child says hello to parent. Parent writes it to stdio.

```
int main(void) {
    int n, fd[2];
    pid_t pid;
    
    char line[maxline];
    if(pipe(fd) < 0)  
        err_sys("pipe error");

    if( (pid = fork()) < 0)  
        err_sys("fork error");
    else if(pid > 0) {
        close(fd[0]);
        write(fd[1], "hello\n", 6);
    } 
    else {
        close(fd[1]);
        n = read(fd[0], line, MAXLINE);
        write(STDOUT_FILENO, line, n);
    }
}
```

- Pipe parent's stdout to child. This is essentially a popen.

```
    int fd[2];
    pid_t pid;
    
    pipe(fd);
    pid = fork();
    if(pid == 0) {
        dup2(fd[0], STDIN_FILENO);
        exec(<whatever>);
    }
```

* popenFILE * popen (const char *command, const char *mode)
    * Creates a pipe
    * Forks
    * Sets up pipe between parent and child (type specifies direction)
    * Closes unused ends of pipes
    * Turns pipes into FILE pointers for use with STDIO functions (fread, fwrite, printf, scanf, etc.)
    * Execs shell to run cmdstring on child

```
#include <stdio.h>
#include <stdlib.h>

void
write_data (FILE * stream)
{
  int i;
  for (i = 0; i < 100; i++)
    fprintf (stream, "%d\n", i);
  if (ferror (stream))
    {
      fprintf (stderr, "Output to stream failed.\n");
      exit (EXIT_FAILURE);
    }
}

int
main (void)
{
  FILE *output;

  output = popen ("more", "w");
  if (!output)
    {
      fprintf (stderr,
               "incorrect parameters or too many files.\n");
      return EXIT_FAILURE;
    }
  write_data (output);
  if (pclose (output) != 0)
    {
      fprintf (stderr,
               "Could not run more or other error.\n");
    }
  return EXIT_SUCCESS;
}

```


