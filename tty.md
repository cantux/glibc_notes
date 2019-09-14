http://www.linusakesson.net/programming/tty/

You are reading from a tty(4) (in the usual case when stdin is your terminal). These are tricky things, read the tty demystified.

Be aware that your terminal and its tty has some line discipline. So, some data is buffered in the kernel (and also in the standard library).

You might want to put your tty in raw mode. See termios(3) & stty(1)

But don't lose your time, instead use some library like ncurses or readline

To play with select, you might use some fifo(7), perhaps with mkfifo /tmp/myfifo then yourprogram < /tmp/myfifo and, in another terminal, echo hello > /tmp/myfifo

```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>

int main(void)
{
    fd_set rfds;
    char buf[100];

    while(1)
    {
        FD_ZERO(&rfds);       
        FD_SET(fileno(stdin), &rfds);

        if(-1 == select(FD_SETSIZE, &rfds, NULL, NULL, NULL))
        {
            perror("select() failed\n");
        }

        if(FD_ISSET(fileno(stdin), &rfds)) 
        {
            printf("Keyboard input received: ");
            fgets(buf, 5, stdin);
            printf("%s\n", buf);
        }
    }
    return 0;
}
```
