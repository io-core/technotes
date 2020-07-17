# Tech Note 004 - Large Files, Unicode Filenames, and Subdirectories
### Modifying the Oberon file system to support large files, long unicode filenames, and subdirectories

* Block Size

The Project Oberon FileDir module operates on 1024 byte (1K) blocks, with the following limits:

    * disk size - 141.2 GiB  ((2^32)/29) x 1k sectors but in practice limited to 64MB due to 2048 word sector bitmap in Kernel.Mod
    * file size - 3 MiB (64+(12*256)) x 1k sectors

With FileDir and Files adjusted for 4k sectors, using 64-bit values the Oberon file system and with on-disk storage of bitmaps to relieve memory pressure, an extended Oberon file system may address a much larger volume:

    * disk size - 2 ZiB  ((2^64)/29) x 4k sectors

If FileDir and Files are adjusted for 4k sectors but accepting a limit of 32-bit values for sector addresses the Oberon file system may store a significant amount of data:

    * disk size - 564.9 GiB  ((2^32)/29) x 4k sectors

File size in Project Oberon FileDir and Files is constrained by the nunber of direct and indirect (SecTab and ExTab) entries in the FileHeader record. File sizes to the limit of the volume size could be supported by specifying that the last entry of any span of a 'sector table' would refer to the beginning of another span of 'sector table' values with one greater level of indirection.

* Page Alignment

Project Oberon files start with a FileHeader structure of 352 bytes, after which 672 bytes of file data may fill out the first file page. Oberon file data is therefore not aligned on disk pages with their offsets as indexed by file position, complicating the implementation of memory mapping of files (which Project Oberon does not do.) On the other hand, files with 672 bytes of data or less do not require indirection to another disk block of storage. 

A more compact FileHeader structe of 64 bytes (without the filename which is already found at the Directory level) and with four (3+1) top level sector table entries allows for 64 FileHeaders per 4096-byte sector and allows byte zero of a file to reside at offset zero of the first sector. Partially full FileHeader sectors may be linked in a chain of 'next sector' entries for efficiency in locating a free FileHeader slot.

* Name Length and encoding

Filedir in Project Oberon allocates 32 bytes for the file name and two 32-bit sector references in each DirEntry. With page expansion to 4k, 40 byte Unicode file names, two 64-bit sector references, and a 64-bit status word will allow for 63 entries per DirPage. A status word may indicate that the space for subsequent entries are used for overflow filename space, allowing for longer filenames.

* Subdirectories

FileDir can be modified to allow a file in a directory to be the root of another directory, thereby implementing subdirectories. Accessing files within subdirectories may require adjustment to other modules that operate on files however as the assumption that all files are in one directory will no longer be valid.

### Adopting 4k sectors

An expanded sector size requires some constants to be modified and makes space for additional features. With 64-bit sector indices (treated as two 32-bit words on a 32-bit system) an expanded Oberon volume can be up to 2 Zetabytes in size.


Oberon Filesystem Feature | Original | Expanded 
-------------------------:|:--------:|----------
Sector Size               |  1024    |   4096
Sector Index              |  32 bit  | 64 bit
Max Volume                | 141.2 GiB| 2 ZiB
Filename Encoding         |  ascii   |  utf8
Filename Length           | 31 bytes |  40+ bytes
Sector Table Entries      | 64       | 3+1
Single Extension Entries  | 12       | na
Maximum File Size         | 3MB      | volume limit
Index Sector Entries      | 256      | 512
File Header Size          | 352      | 64
Root Directory Address    | 29       | 29
Directory Entries per Page| 24       | 63
Directory Mark            | 9B1EA38D |9B1EA38E
File Header Mark          | 9BA71D86 | na
File Header FillerSize    | 52       | na

More detail to go here...
