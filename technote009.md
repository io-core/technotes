# Tech Note 009 - 64-bit Oberon0
### Adapting the Oberon07 compiler and Oberon OS to 64-bits 

* Much has been done before: https://www.modulaware.com/mdlt68.htm https://github.com/vishaps/voc
* Same compiler source should compile on 64 and 32 bit systems
* 64 and 32 bit compilers should compile 64 and 32 bit code equivalently
* LONGINT should accept a system pointer, therefore follows system register size

### Extending RISC5 to 64-bits (erisc5)

* registers R0 through R15, PC, H, SPC extended from 32-bit to 64-bit
* 32-bit opcodes unchanged
* 64-bit word loads and stores alingned to 32-bit boundaries
* 64-bit memory-mapped device registers take double the space

### General OS considerations for 64-bit

* start of general use system RAM may not be 0 (0x00200000 on Pinebook Pro, for example)
* Method Table entries for 64-bit Oberon take 8 bytes instead of 4 bytes for 32-bit Oberon
* Method Table start at start of system RAM  + 64 bytes for 64-bit Oberon
* In 32-bit RISC5 Oberon there is space for 31 MT entries (through byte address 255.)
* 64-bit Oberon may reserve space for 1008 entries (up to Start of System RAM + 8192 bytes.)
* 32-bit Oberon may also reserve space for 1008 entries (up to Start + 4096 bytes.)
* First 32 or 64 bytes are implementation dependent and reserved (e.g. for reset vector, etc.)
* Kernel.Mod for each platform is platform dependent and will define interrupt vectors, mmu tables, etc. as appropriate.
* On hardware providing u-boot, Kernel.Mod may retrieve useful configuration information from a Device Tree Blob.
* On hardware providing UEFI, Kernel.Mod may retrieve useful information from the firmware.

