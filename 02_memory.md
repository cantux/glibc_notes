Virtual memory is divided into pages. Kernel is responsible for handling page faults. You can lock pages to have them loaded in the memory. Shiiiiiiieeet

Processes allocate memory in three major ways: 

- exec 
- fork
- programatically 
- memory mapped I/O.

exec creates a virtual address space within the program. 

See [Unconstrained Allocation](https://www.gnu.org/software/libc/manual/html_node/Unconstrained-Allocation.html#Unconstrained-Allocation)

One interesting thing is the fnc ptrs to malloc, realloc calls.







