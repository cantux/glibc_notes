### POSIX Safety Concepts

MT: MultiThread, AS: Async Signal, AC: Async Cancel -Safety

*Preliminary*: These properties are documented but they may not be included in the future releases of GNU C Library Interface.
It is intended to remove the preliminaries from the library.

*MT-Safe*: Function is safe to call in presence of other threads. Does not imply a atomicity. Does not imply that memory safety mechanisms of POSIX is used.

*AS-Safe*: Safe to call from asyn signal handlers.

*AC-Safe*: Safe to call from pthread_cancel and its ilk's handler.


See below for further detailed concepts on safety:

https://www.gnu.org/software/libc/manual/html_node/Unsafe-Features.html#Unsafe-Features
