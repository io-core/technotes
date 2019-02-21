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

Oberon Filesystem Feature | Original | Expanded 
-------------------------:|:--------:|----------
Sector Size               |  1024    |   4096
Sector Index              |  32 bit  | 64 bit
Max Volume                | 141.2 GiB| 2 ZiB
Filename Encoding         |  ascii   |  utf8
Filename Length           | 31 bytes |  127 bytes
Sector Table Entries      | 64       | 64
Extension Table Entries   | 12       | 64
Maximum File Size         | 3MB      | 16 TiB (64 + 12*512 + 16*512*512 + 16*512*512*512)
Index Sector Entries      | 256      | 512

```
// 32-bit oberon file system  141.2 GiB Max Volume          // 64-bit oberon filesystem   2 ZiB Max Volume
const RFS_FnLength    = 32                                  // 127 + a zero byte = 128
const RFS_SecTabSize  = 64                                  // 64 -- 64-bit integers mod 29
const RFS_ExTabSize   = 12  //64+12*256 = 3MB max file size // 64 + 12*512 + 16*512*512 + 16*512*512*512 = 16 TiB max file size
const RFS_SectorSize  = 1024                                // 4096
const RFS_IndexSize   = 256    //SectorSize / 4             // 512  -- SectorSize / 8
const RFS_HeaderSize  = 352                                 // ??
const RFS_DirRootAdr  = 29                                  // 29
const RFS_DirPgSize   = 24                                  // 24
const RFS_N = 12               //DirPgSize / 2              // 12
const RFS_DirMark    = 0x9B1EA38D                           // 0x9B1EA38E
const RFS_HeaderMark = 0x9BA71D86                           // 0x9BA71D87
//  RFS_MERKLEHASH                                          // SHA256 hash of: filenames + hashes of file contents of all files in directory
const RFS_FillerSize = 52                                   // ??

```

More detail to go here...
