### File Attributes

```
#include sys/stat.h 

struct stat {

    // Specifies the mode of the file. This includes file type information (see Testing File Type) and the file permission bits (see Permission Bits).
    mode_t st_mode

    // The file serial number, which distinguishes this file from all other files on the same device.
    ino_t st_ino

    // Identifies the device containing the file. The st_ino and st_dev, taken together, uniquely identify the file.
    //  The st_dev value is not necessarily consistent across reboots or system crashes, however.
    dev_t st_dev

    // The number of hard links to the file. This count keeps track of how many directories have entries for this file. If the count is ever decremented to zero, then the file itself is discarded as soon as no process still holds it open. Symbolic links are not counted in the total.
    nlink_t st_nlink

    // The user ID of the file's owner. See File Owner.
    uid_t st_uid

    // The group ID of the file. See File Owner.
    gid_t st_gid

    // This specifies the size of a regular file in bytes. For files that are really devices this field isn't usually meaningful. For symbolic links this specifies the length of the file name the link refers to.
    off_t st_size

    time_t st_atime                 // This is the last access time for the file. See File Times.
    unsigned long int st_atime_usec // This is the fractional part of the last access time for the file. See File Times.
    time_t st_mtime                 // This is the time of the last modification to the contents of the file. See File Times.
    unsigned long int st_mtime_usec // This is the fractional part of the time of the last modification to the contents of the file. See File Times.
    time_t st_ctime                 // This is the time of the last modification to the attributes of the file. See File Times.
    unsigned long int st_ctime_usec // This is the fractional part of the time of the last modification to the attributes of the file. See File Times.

    // The number of disk blocks is not strictly proportional to the size of the file, for two reasons: 
    //          the file system may use some blocks for internal record keeping; 
    //          the file may be sparse--it may have "holes" which contain zeros but do not actually take up space on the disk.
    blkcnt_t st_blocks

    // The optimal block size for reading or writing this file, in bytes. 
    // You might use this size for allocating the buffer space for reading or writing the file. (This is unrelated to st_blocks.) 
    unsigned int st_blksize
}


int stat (const char *filename, struct stat *buf)

// does symlinks have filedescs? if not, does fstat follow symlinks? 
// Since difference is not mentioned in the documentation I imagine they do not. 
// Because otherwise we would need an flstat fnc too right?
int fstat (int filedes, struct stat *buf)

// like stat, doesnt follow symlinks
int lstat (const char *filename, struct stat *buf)
```
