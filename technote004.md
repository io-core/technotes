# Tech Note 004 - Large Files, Unicode Filenames, and Subdirectories
### Modifying the Oberon file system to support large files, long unicode filenames, and subdirectories

The Project Oberon FileDir module operates on 1024 byte (1K) blocks, with the following limits:

    * disk size - 141.2 GiB  ((2^32)/29) x 1k sectors but in practice only 64MB due to a limit in Kernel.Mod
    * file size - 3 MiB (64+(12*256)) x 1k sectors

With FileDir and Files adjusted for 4k sectors, using 64-bit values the Oberon file system and with on-disk storage of bitmaps to relieve memory pressure, and with an open-ended chain of expanding indirection for finding the nth file block, an extended Oberon file system may address a much larger volume:

    * disk size - 2 ZiB  ((2^64)/29) x 4k sectors or 564.9 GiB with 32-bit sector values ((2^32)/29) x 4k sectors
    * file size - limited to partition size

The following constants in a modified FileDir.Mod identify key aspects of an Oberon Extended File System largely based on the original Oberon File System:

```

  CONST FnLength*    = 47;
        TabSize*     = 4;
        SectorSize*  = 4096;
        IndexSize*   = SectorSize DIV 8;
        HeaderSize*  = 64;
        DirRootAdr*  = 29;
        DirPgSize*   = 63;
        HdrPgSize*   = 63;
        N = DirPgSize DIV 2;
        DirMark*    = 9B1EA38EH;
        HeaderMark* = 9BA71D87H;
        FillerSize = 40;
        
```

A 64-bit filesystem in a 32-bit operating system

It would be helpful to be able to use the same on-disk structures with either a 32-bit Project Oberon or a 64-bit one. This technote will describe changes to the 32-bit project oberon to use a filesystem with limits that would be relaxed or simplified in a true 64-bit implememtation. For example, disk addresses will be structs or arrays of two 32-bit values rather than simple 64-bit ones, as shown here:

```

TYPE 
    Val64*         = 
      RECORD
          low* : INTEGER ;
          high*: INTEGER ;
      END ;
    DiskAdr*        = Val64 ;
    FileName*       = ARRAY FnLength OF CHAR;
    SectorTable*    = ARRAY TabSize OF DiskAdr;
    EntryHandler*   = PROCEDURE (name: FileName; sec: DiskAdr; VAR continue: BOOLEAN);

    FileHeader* =
      RECORD 
        kind*: INTEGER; (* File Type *)
        perm*: INTEGER; (* File Permissions *)
        date*: Val64; (* File Modification Timestamp *)
        length*: Val64; 
        owner*: INTEGER;
        group*: INTEGER;
        SecTab: ARRAY TabSize OF Val64;
      END ;

```

Project Oberon files start with a FileHeader structure of 352 bytes, after which 672 bytes of file data may fill out the first file page. Oberon file data is therefore not aligned on disk pages with their offsets as indexed by file position, complicating the implementation of memory mapping of files (which Project Oberon does not do.) On the other hand, files with 672 bytes of data or less do not require indirection to another disk block of storage. 

A more compact FileHeader structe of 64 bytes (without the filename which is already found at the Directory level) and with four (3+1) top level sector table entries allows for 64 FileHeaders per 4096-byte sector and allows byte zero of a file to reside at offset zero of the first sector. Partially full FileHeader sectors may be linked in a chain of 'next sector' entries for efficiency in locating a free FileHeader slot.

* Name Length and encoding

Filedir in Project Oberon allocates 32 bytes for the file name and two 32-bit sector references in each DirEntry. With page expansion to 4k, 47 byte Unicode file names, two 64-bit sector references, and a byte for the HeaderPage index (6 bits) and pre-empting subsequent DirEntries for a longer file name (2 bits) will allow for 63 entries per DirPage and file names up to 303 bytes in length, which may be encoded UTF8.

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
Filename Length           | 31 bytes |  47+ bytes
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
