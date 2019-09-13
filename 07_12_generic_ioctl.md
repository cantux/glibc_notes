### Generic I/O Control operations

GNU systems can handle most input/output operations on many different devices and objects in terms of a few file primitives - read, write and lseek. However, most devices also have a few peculiar operations which do not fit into this model. Such as:

- Changing the character font used on a terminal.
- Telling a magnetic tape system to rewind or fast forward. (Since they cannot move in byte increments, lseek is inapplicable).
- Ejecting a disk from a drive.
- Playing an audio track from a CD-ROM drive.
- Maintaining routing tables for a network. 

Although some such objects such as sockets and terminals 3 have special functions of their own, it would not be practical to create functions for all these cases.

Instead these minor operations, known as IOCTLs, are assigned code numbers and multiplexed through the ioctl function, defined in sys/ioctl.h. The code numbers themselves are defined in many different headers. 
