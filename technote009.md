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
