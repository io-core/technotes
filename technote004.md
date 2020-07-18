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

It would be helpful to be able to use the same on-disk structures with either a 32-bit Project Oberon or a 64-bit one. This technote will describe changes to the 32-bit project oberon to use a filesystem with limits that would be relaxed or simplified in a true 64-bit implememtation. For example, disk addresses will be structs or arrays of two 32-bit values rather than simple 64-bit ones, as shown here:

```

TYPE 
    Val64* = 
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
        sectab: ARRAY TabSize OF Val64;
      END ;

      HeaderPage* =
        RECORD
          mark*: INTEGER;
          zero*: INTEGER;
          next*: Val64;       
          fill*: ARRAY 48 OF CHAR;
          hdr*: ARRAY HdrPgSize OF FileHeader;
        END ;
```

The FileHeader and HeaderPage above are a departure from the original Oberon file system which located the FileHeader at the start of a file. The above scheme allows file data to begin at offset zero of underlying disk sectors in the storage medium, simplifying the mapping of files into memory should that be desired by an operating system implemeting the file system. In the above scheme the FileHeaders are collected into HeaderPages which include a Next field pointing to a next non-full HeaderPage sector to assist recycling of FileHeader entries. Also in the above system filenames are not stored in FileHeaders, allowing the implementation of symbolic and hard links, should those be desired.

Another departure of the above scheme is the SecTab array of the FileHeader structure containing only four entries in which the last entry is reserved to point to an array (page) of 511 values with one greater degree of indirection and one value (the 512th) recapitulating the scheme a further degree of indirection. 255T can be addressed with three levels of indirection and the scheme can be extended indefinitely.

```
12K  with no indirection:   4096 * 3 +  
2M   with 1 indirection:    4096 * 511 +
1G   with 2 indirections:   4096 * 512 * 511 +
255T with 3 indirections:   4096 * 512 * 512 * 511 +
```

The Oberon B-Tree of directory entries remains but with a wider fan-out due to a larger sector size and the potential for longer file names:

```

    DirEntry* =  (*B-tree node*)
      RECORD
        p*:    DiskAdr;  (*sec no of descendant in directory*)
        adr*:  DiskAdr;  (*sec no of file header*)
        i*:    CHAR;     (*low 6 bits of I are the HeaderPage index to be used with the adr field *)
        name*: FileName  (*top 2 bits of I are number of subsequent DirEntry slots pre-empted *)
      END ;              (*for a longer filename for this entry for a maximum name length of 303 bytes *)

    DirPage*  =
      RECORD mark*:  INTEGER;
        m*:     INTEGER;
        zero:   INTEGER;
        p0*:    DiskAdr;  (*sec no of left descendant in directory*)
        fill:  ARRAY FillerSize OF BYTE;
        e*:  ARRAY DirPgSize OF DirEntry
      END ;

```

* block allocation tables

A small filesystem may not implement a block allocation table bitmap on disk and merely scan all the DirPages and FileHeaders on startup but for a very large filesystem this would cause excessive delays and would use a large amount of memory.

Since a 4K sector contains 32768 bits a storage medium may be divided into chunks of 32768 sectors with the last sector in each chunk reserved for the bitmap page.

* Subdirectories

The 'kind' field in the FileHeader record may indicate that the entry is for a sub-directory rather than a file, in which case sectab[0] will point to a DirPage rather than a file data sector. Making use of files within subdirectories will require adjustment to other modules that operate on files as the assumption that all files are in one directory will no longer be valid. Making use of filenames longer than 32 bytes and containing Unicode characters will require similar changes.
