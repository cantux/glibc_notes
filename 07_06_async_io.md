### I/O Operations in Parallel

Initiate one or more I/O operations and then immediately resume normal work while the I/O operations are executed in parallel. 

```
struct aiocb64 {
    int aio_fildes;         // filedes used for the operation
    off64_t aio_offset;
    volatile void *aio_buf; // pointer to the buffer with the data to be written
    size_t aio_nbytes;      // length of the aio buffer
    
    // If for the platform _POSIX_PRIORITIZED_IO and _POSIX_PRIORITY_SCHEDULING are defined the AIO requests are processed based on the current scheduling priority. i
    // The aio_reqprio element can then be used to lower the priority of the AIO operation. 
    int aio_reqprio;        

    // This element specifies how the calling process is notified once the operation terminates. 
    // If the sigev_notify element is SIGEV_NONE no notification is sent. If it is SIGEV_SIGNAL, the signal determined by sigev_signo is sent. 
    // Otherwise, sigev_notify must be SIGEV_THREAD in which case a thread is created which starts executing the function pointed to by sigev_notify_function. 
    struct sigevent aio_sigevent

    // Only used by the lio_listio and lio_listio64 functions.
    // Since these functions allow an arbitrary number of operations to start at once, and each operation can be input or output (or nothing), the information must be stored in the control block. The possible values are:
    // LIO_READ: Start a read operation. Read from the file at position aio_offset and store the next aio_nbytes bytes in the buffer pointed to by aio_buf.
    // LIO_WRITE: Start a write operation. Write aio_nbytes bytes starting at aio_buf into the file starting at position aio_offset.
    // LIO_NOP: Do nothing for this control block. This value is useful sometimes when an array of struct aiocb values contains holes, i.e., some of the values must not be handled although the whole array is presented to the lio_listio function. 
    int aio_lio_opcode

} 

   #include <signal.h>

   union sigval {          /* Data passed with notification */
       int     sival_int;         /* Integer value */
       void   *sival_ptr;         /* Pointer value */
   };

   struct sigevent {
       int          sigev_notify; /* Notification method */
       int          sigev_signo;  /* Notification signal */
       union sigval sigev_value;  /* Data passed with
                                     notification */
       void       (*sigev_notify_function) (union sigval);
                        /* Function used for thread
                           notification (SIGEV_THREAD) */
       void        *sigev_notify_attributes;
                        /* Attributes for notification thread
                           (SIGEV_THREAD) */
       pid_t        sigev_notify_thread_id;
                        /* ID of thread to signal (SIGEV_THREAD_ID) */
   };

// initiates an asynchronous read operation. It immediately returns after the operation was enqueued or when an error was encountered. 
// The calling process is notified about the termination of the read request according to the aiocbp->aio_sigevent value. 
int aio_read (struct aiocb *aiocbp)
int aio_read64 (struct aiocb64 *aiocbp)

int aio_write (struct aiocb *aiocbp)
int aio_write64 (struct aiocb64 *aiocbp)


// The lio_listio function can be used to enqueue an arbitrary number of read and write requests at one time. 
// The requests can all be meant for the same file, all for different files or every solution in between. 
// LIO_WAIT, LIO_NOWAIT
int lio_listio (int mode, struct aiocb *const list[], int nent, struct sigevent *sig)

int aio_error (const struct aiocb *aiocbp)
ssize_t aio_return (struct aiocb *aiocbp)
int aio_fsync (int op, struct aiocb *aiocbp)
int aio_suspend (const struct aiocb *const list[], int nent, const struct timespec *timeout)
int aio_cancel (int fildes, struct aiocb *aiocbp)
```

```
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
#include <aio.h>
#include <signal.h>

#define BUF_SIZE 20     /* Size of buffers for read operations */

#define errExit(msg) do { perror(msg); exit(EXIT_FAILURE); } while (0)

#define errMsg(msg)  do { perror(msg); } while (0)

struct ioRequest {      /* Application-defined structure for tracking
                          I/O requests */
   int           reqNum;
   int           status;
   struct aiocb *aiocbp;
};

static volatile sig_atomic_t gotSIGQUIT = 0;
                       /* On delivery of SIGQUIT, we attempt to
                          cancel all outstanding I/O requests */

static void             /* Handler for SIGQUIT */
quitHandler(int sig)
{
   gotSIGQUIT = 1;
}

#define IO_SIGNAL SIGUSR1   /* Signal used to notify I/O completion */

static void                 /* Handler for I/O completion signal */
aioSigHandler(int sig, siginfo_t *si, void *ucontext)
{
   if (si->si_code == SI_ASYNCIO) {
       write(STDOUT_FILENO, "I/O completion signal received\n", 31);

       /* The corresponding ioRequest structure would be available as
              struct ioRequest *ioReq = si->si_value.sival_ptr;
          and the file descriptor would then be available via
              ioReq->aiocbp->aio_fildes */
   }
}

int
main(int argc, char *argv[])
{
   struct ioRequest *ioList;
   struct aiocb *aiocbList;
   struct sigaction sa;
   int s, j;
   int numReqs;        /* Total number of queued I/O requests */
   int openReqs;       /* Number of I/O requests still in progress */

   if (argc < 2) {
       fprintf(stderr, "Usage: %s <pathname> <pathname>...\n",
               argv[0]);
       exit(EXIT_FAILURE);
   }

   numReqs = argc - 1;

   /* Allocate our arrays */

   ioList = calloc(numReqs, sizeof(struct ioRequest));
   if (ioList == NULL)
       errExit("calloc");

   aiocbList = calloc(numReqs, sizeof(struct aiocb));
   if (aiocbList == NULL)
       errExit("calloc");

   /* Establish handlers for SIGQUIT and the I/O completion signal */

   sa.sa_flags = SA_RESTART;
   sigemptyset(&sa.sa_mask);

   sa.sa_handler = quitHandler;
   if (sigaction(SIGQUIT, &sa, NULL) == -1)
       errExit("sigaction");

   sa.sa_flags = SA_RESTART | SA_SIGINFO;
   sa.sa_sigaction = aioSigHandler;
   if (sigaction(IO_SIGNAL, &sa, NULL) == -1)
       errExit("sigaction");

   /* Open each file specified on the command line, and queue
      a read request on the resulting file descriptor */

   for (j = 0; j < numReqs; j++) {
       ioList[j].reqNum = j;
       ioList[j].status = EINPROGRESS;
       ioList[j].aiocbp = &aiocbList[j];

       ioList[j].aiocbp->aio_fildes = open(argv[j + 1], O_RDONLY);
       if (ioList[j].aiocbp->aio_fildes == -1)
           errExit("open");
       printf("opened %s on descriptor %d\n", argv[j + 1],
               ioList[j].aiocbp->aio_fildes);

       ioList[j].aiocbp->aio_buf = malloc(BUF_SIZE);
       if (ioList[j].aiocbp->aio_buf == NULL)
           errExit("malloc");

       ioList[j].aiocbp->aio_nbytes = BUF_SIZE;
       ioList[j].aiocbp->aio_reqprio = 0;
       ioList[j].aiocbp->aio_offset = 0;
       ioList[j].aiocbp->aio_sigevent.sigev_notify = SIGEV_SIGNAL;
       ioList[j].aiocbp->aio_sigevent.sigev_signo = IO_SIGNAL;
       ioList[j].aiocbp->aio_sigevent.sigev_value.sival_ptr =
                               &ioList[j];

       s = aio_read(ioList[j].aiocbp);
       if (s == -1)
           errExit("aio_read");
   }

   openReqs = numReqs;

   /* Loop, monitoring status of I/O requests */

   while (openReqs > 0) {
       sleep(3);       /* Delay between each monitoring step */

       if (gotSIGQUIT) {

           /* On receipt of SIGQUIT, attempt to cancel each of the
              outstanding I/O requests, and display status returned
              from the cancellation requests */

           printf("got SIGQUIT; canceling I/O requests: \n");

           for (j = 0; j < numReqs; j++) {
               if (ioList[j].status == EINPROGRESS) {
                   printf("    Request %d on descriptor %d:", j,
                           ioList[j].aiocbp->aio_fildes);
                   s = aio_cancel(ioList[j].aiocbp->aio_fildes,
                           ioList[j].aiocbp);
                   if (s == AIO_CANCELED)
                       printf("I/O canceled\n");
                   else if (s == AIO_NOTCANCELED)
                       printf("I/O not canceled\n");
                   else if (s == AIO_ALLDONE)
                       printf("I/O all done\n");
                   else
                       errMsg("aio_cancel");
               }
           }

           gotSIGQUIT = 0;
       }

       /* Check the status of each I/O request that is still
          in progress */

       printf("aio_error():\n");
       for (j = 0; j < numReqs; j++) {
           if (ioList[j].status == EINPROGRESS) {
               printf("    for request %d (descriptor %d): ",
                       j, ioList[j].aiocbp->aio_fildes);
               ioList[j].status = aio_error(ioList[j].aiocbp);

               switch (ioList[j].status) {
               case 0:
                   printf("I/O succeeded\n");
                   break;
               case EINPROGRESS:
                   printf("In progress\n");
                   break;
               case ECANCELED:
                   printf("Canceled\n");
                   break;
               default:
                   errMsg("aio_error");
                   break;
               }

               if (ioList[j].status != EINPROGRESS)
                   openReqs--;
           }
       }
   }

   printf("All I/O requests completed\n");

   /* Check status return of all I/O requests */

   printf("aio_return():\n");
   for (j = 0; j < numReqs; j++) {
       ssize_t s;

       s = aio_return(ioList[j].aiocbp);
       printf("    for request %d (descriptor %d): %zd\n",
               j, ioList[j].aiocbp->aio_fildes, s);
   }

   exit(EXIT_SUCCESS);
}
```
