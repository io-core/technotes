# Tech Note 004 - Large Files, Unicode Filenames, and Subdirectories
### Modifying the Oberon file system to support large files, long unicode filenames, and subdirectories

* Block Size

The Project Oberon FileDir module operates on 1024 byte (1K) blocks, with the following limits:

    * disk size - 141.2 GiB  ((2^32)/29) x 1k sectors
    * file size - 3 MiB (64+(12*256)) x 1k sectors

With FileDir and Files adjusted for 4k sectors and using 64-bit values as discussed below, the Oberon file system may address:

    * disk size - 2 ZiB  ((2^64)/29) x 4k sectors
    * file size - 16TiB  (64+(12*512)+(16*512*512)+(16*512*512*512)) x 4k sectors

* Name Length and encoding

Filedir in Project Oberon allocates 32 bytes for the file name and two 32-bit sector references in each DirEntry. With page expansion to 4k, 128 byte Unicode file names and two 64-bit sector references will allow for the same 24 entries per DirPage.

* Subdirectories

FileDir can be modified to allow a file in a directory to be the root of another directory, thereby implementing subdirectories. Accessing files within subdirectories may require adjustment to other modules that operate on files however as the assumption that all files are in one directory will no longer be valid.

### Adopting 4k sectors

More detail to go here...
