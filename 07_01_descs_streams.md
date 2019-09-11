## Low Level I/O - Descriptors and Streams

```
// Create a stream from a fd. See open types of streams.
FILE * fdopen (int filedes, const char *opentype)

// Get the fd from stream
int fileno (FILE *stream)

// why not just call this unix std??
#include unistd.h 
// definitions for standard streams
STDIN_FILENO, STDOU_FILENO, STDERR_FILENO
```

### Precautions

You can have multiple channels(descriptors and streams) connected to the same file. There are two cases to consider:

#### Linked Channels

Share a single file position value. Recommended except when all the access is for input??

- Make a stream with fdopen
- Get a desc with fileno
- dup
- fork
- append-type output streams
- non-random access file types like Pipes and Terminals

If you open a pipe, something that can only be done at the descriptor level, either do all I/O with the descriptor 
or construct a stream from the desc with fdopen.

!!! For files that don't support random access, all channels are linked
!!! For files that support random access, all *append* type *output* stream are linked.
!!! If you have been using a stream for I/O and you want to do I/O using another channel you must first clean up the stream.
!!! Terminating a process or executing a new program in the process destroys all the streams in the process. 
        if descriptors linked to these streams persist their file positions become undefined. Clean up to prevent this!


#### Independent Channels

When open independently, channels have their own position.

- Clean an output stream after use, before doing anything else that might read or write from the same part of the file.
- Clean an input stream before reading data that may have been modified using an independent channel. Otherwise, you might read obselete data from stream's buffer.

If first channel outputs to the end of file, second channel has no reliable way to set the position to the end of the file, because the first can always go in append something new.

Instead, use an append-type descriptor or stream; they always output at the current end of the file.

If it is a stream you must clean in order to make the end-of-file position accurate.

#### Cleaning Streams

```
fflush
```

No need to use a stream if it is clean:

- Unbuffered stream is always clean.
- An Input stream that is at EOF is clean.
- line-buffered stream is clean when the last character output was a newline.
- just-opened input stream might not be clean, as its input buffer might not ne empty.


Closing an output stream automatically calls fflush.

There is one case in which cleaning a stream is impossible on most systems.
This is when the stream is doing input from a file that is not random-access.
Such streams typically read ahead, and when the file is not random access, there is no way to give back the excess data already read. 
When an input stream reads from a random-access file, fflush does clean the stream, but leaves the file pointer at an unpredictable place; you must set the file pointer before doing any further I/O. 

You need not clean a stream before using its descriptor for control operations such as setting terminal modes; 
these operations don't affect the file position and are not affected by it. You can use any descriptor for these operations, and all channels are affected simultaneously. 
However, text already "output" to a stream but still buffered by the stream will be subject to the new terminal modes when subsequently flushed. 
To make sure "past" output is covered by the terminal settings that were in effect at the time, flush the output streams for that terminal before setting the modes. See Terminal Modes. 


