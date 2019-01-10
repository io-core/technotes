# Tech Note 002 - Adjusting Module, Stack, Heap, and Video Memory
## Adjusting RISC5 Oberon memory use

The Oberon V5 system receives `MemLim` at memory address `12` with the value `E7EF0H` and `stackOrg` at memory address `24` with the value `800000H` from the Boot Loader. From these two values it calculates its module, stack, and heap reserved locations. The start of video memory is statically defined to be `E7F00H` (which happens to be equivalent to MemLim + 16) and the video geometry is statically defined to be monochrome 1024x768 with 128 bytes per 'scan line' in `Display.Mod`.

Oberon can be modified to use more memory for the display, the heap, and for module space by 1) Modifying Display.Mod for other screen geometries, 2) Patching or modifying the bootloader, and 3) Modifying Display.Mod to account for a different base address. 

### Modifying Display.Mod for dynamic screen geometries

Other geometries than 1024x768 can be supported by modifying only Display.Mod (and, of course, the hardware or emulator of the Oberon RISC5 system.) The simplest and most direct method is to adjust the constants in Display.Mod to reflect the changed width and height and color depth and base offset of the screen. 

If the screen resolution is not constant then Display.Mod needs to acquire the geometry at run-time. One method is to use a convention of placing the width of the screen at location `base+4`, the height of the screen at location `base+8` and a constant, e.g. `53697A66H` at the `base` of the screen buffer memory. Display.Mod may these values and if it finds the constant then it adopts the width and height values, otherwise it uses the default values of 1024x768(x1).  Alternative methods might include reading hardware registers or obtaining values from the boot loader like is done with stackOrg and MemLim.

As the base offset for the screen has not changed, the code may still use the constant `E7F00` in its drawing calculations. The Display.Mod code can no longer use the constant `128` as the number of bytes that contain a row of pixels on the screen.
 An updated Display.Mod must calculate this at initialization time and use the variable instead of a constant at runtime.
 
### Patching or modifying the Bootloader to provide more memory to the Oberon system

Providing more module and heap memory to the RISC5 Oberon system does not require changes to any module other than Display.Mod, but does require changes to the boot loader, as it is the responsibility of the boot loader to deposit the MemLim and StackOrg values in memory after loading the system image.

An Oberon environment with significantly different hardware can be expected to have a custom boot loader for loading Oberon on that hardware That boot loader will place the appropriate MemLim and StackOrg values in memory addresses 12 and 24 respectively.

A RISC5 Oberon emulator where the only real difference is more memory, however, might wish to re-use the existing boot firmware image. When the bootloader is placed in emulated ROM, word offsets 372 and 373 encode the loading of a register with memLim value that is stored to location 12 and word offset 376 encodes the loading of a register with the StackOrg value that is stored to location 24. They might be updated as follows: `ROM[372]=0x61000000+(newMLim >> 16); ROM[373]=0x41160000+(newMLim & 0x0000FFFF); ROM[372]=0x61000000+(newStackOrg >> 16);` which assumes that the lower 16 bits of newStackOrg are zero. 

### Modifying Display.Mod to account for a different base address

If MemLim has changed then Display.Mod must also be changed to reflect the new display base address. The simplest way to adapt to the new base is to simply change the `base` constant in Display.Mod and recompile, as above with changed geometry. If the Display module is expected to cope with different base addresses however, and the hardware or software places video memory after module and heap memory, then Display.Mod can be changed to use the `Base` variable in its calculations instead of the `base` constant. `Base` in this case can be found by adding 16 to the MemLim value found at offset 12 in memory.

The code at https://raw.githubusercontent.com/io-core/io/master/core/Display.Mod uses this method.

