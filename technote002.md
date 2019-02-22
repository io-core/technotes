# Tech Note 002 - Adjusting Module, Stack, Heap, and Video Memory
## Adjusting RISC5 Oberon memory use

The Oberon V5 system receives `MemLim` at memory address `12` with the value `E7EF0H` and `stackOrg` at memory address `24` with the value `800000H` from the Boot Loader. From these two values it calculates its module, stack, and heap reserved locations. The base address of video memory is statically defined to be `E7F00H` (which happens to be equivalent to MemLim + 16) and the video geometry is statically defined using constants in `Display.Mod` to be monochrome 1024x768 with 128 bytes per 'scan line'.

Oberon can be modified to use more memory for the display, the heap, and for module space by 1) Modifying Display.Mod for other screen geometries, 2) Patching or modifying the bootloader, 3) Modifying Display.Mod to account for a different base address, and 4) Modifying Kernel.Mod and Modules.Mod to adjust the stack size.

Here's a diagram of the memory layout in Oberon V5 (note that address zero is at the top of the diagram):

<img src="https://github.com/io-core/technotes/raw/master/OberonMemoryLayout.png">

### Modifying Display.Mod for other screen geometries

Other geometries than 1024x768x1 can be supported by modifying Display.Mod (and, of course, the hardware or emulator of the Oberon RISC5 system.) The simplest and most direct method is to adjust the constants in Display.Mod to reflect the changed width and height and color depth and base offset of the screen. [Display.Mod](https://raw.githubusercontent.com/schierlm/OberonEmulator/master/Oberon/Display.Mod.16Colors.txt) in Michael Schierl's Javascript [OberonEmulator](http://schierlm.github.io/OberonEmulator/emu.html?image=ColorDiskImage&width=800&height=400) uses this method but also adjusts MemLim to allocate more ram to the screen as per the next section.

Alternatively, Display.Mod can be modified to take the height, width, and even color depth as parameters. One method places the width of the screen at location `base+4`, the height of the screen at location `base+8` and the constant `53697A66H` at the `base` address of the screen buffer memory. [Display.Mod](https://raw.githubusercontent.com/pdewacht/oberon-risc-emu/master/Mods/Display.Mod) in Peter De Wachter's [oberon-risc-emu](https://github.com/pdewacht/oberon-risc-emu) uses this method. Other schemes might include reading hardware registers or obtaining values from the boot loader.

The original Oberon RISC5 system had only 1MB and so the following constraints must be satisfied for a system                board.RAM[((newMlim+16)/4)] = 0x53697A66  // magic value 'SIZE'+1
               board.RAM[((newMlim+16)/4)] = 0x53697A66  // magic value 'SIZE'+1
where only the video geometry is changed:  `physMem = 100000H (* 1MB *); base := physMem - 256 - LINELEN * screenH; MemLim := base - 16` (`LINELEN` is screenW / 8, MemLim must end up being `E7EF0` if the boot loader remains unmodified.)

Changing the bits-per-pixel to a value other than `1` (monochrome) introduces more changes in the modules that call Display directly. For example, `white` may no longer be `1` but `15`  in a 4-bit color scheme, or `255` in 8-bit monochrome, or the three bytes `255,255,255` for 24-bit color. Additional variables such as `fgcolor` and `bgcolor` are appropriate. In io-core, [Display.Mod](https://github.com/io-core/io/blob/master/core/Display.Mod) reflects some changes needed for handling color and greyscale.

### Patching or modifying the Bootloader to provide more memory to the Oberon system

Providing more module and heap memory to the RISC5 Oberon system does not require changes to any module other than Display.Mod, but does require changes to the boot loader, as it is the responsibility of the boot loader to deposit the MemLim and StackOrg values in memory after loading the system image.

An Oberon environment with significantly different hardware can be expected to have a custom boot loader for loading Oberon on that hardware. Such a boot loader would determine the appropriate MemLim and StackOrg values and place them in memory addresses 12 and 24 respectively.

A RISC5 Oberon emulator where the only real difference is more memory, however, might wish to re-use the existing boot firmware image. When the bootloader is placed in emulated ROM, word offsets 372 and 373 encode the loading of a register with memLim value that is stored to location 12 and word offsets 340 and 376 encode the loading of registers with the StackOrg value that is used to set up the stack and is stored to location 24. They might be updated as follows: `ROM[340]=0x6e000000+(newStackOrg >> 16); ROM[372]=0x61000000+(newMLim >> 16); ROM[373]=0x41160000+(newMLim & 0x0000FFFF); ROM[376]=0x61000000+(newStackOrg >> 16);` which assumes that the lower 16 bits of newStackOrg are zero. Michael Schierl's fork of oberon-risc-emu [oberon-risc-emu-enhanced](https://github.com/schierlm/oberon-risc-emu-enhanced) works like this.

### Modifying Display.Mod to account for a different base address

If MemLim has changed then Display.Mod must also be changed to reflect the new display base address. The simplest way to adapt to the new base is to simply change the `base` constant in Display.Mod and recompile, as above with changed geometry. 

If the Display module is expected to cope with different base addresses without further recompiling, then Display.Mod can be changed to use a variable in its calculations instead of the `base` constant. The value for the variable can be delivered to Display.Mod as with the geometry ([Display.Mod](https://raw.githubusercontent.com/schierlm/oberon-risc-emu-enhanced/master/Mods/Display.Mod) in oberon-risc-emu-enhanced uses this method), or the base of video can be expected to be found just after the memLim value communicated by the boot loader, as in the io Risc EMulator (REM) [Display.Mod](https://raw.githubusercontent.com/io-core/io/master/core/Display.Mod) which uses this method.

### Modifying Kernel.Mod and Modules.Mod to adjust the stack size

The Oberon files Kernel.Mod and Modules.Mod define the stack to be a fixed 8000H (32k) located just below StackOrg. Patching or recompiling the boot loader to adjust the memory provided to Oberon does not adjust the stack size, only the location. Oberon can be given a larger stack by modifying Kernel.Mod and Modules.Mod with updated values which may be constant or may be calculated at boot time, such as `stackSize := 8000H * (stackOrg >>19)` which would provide, for example, twice the standard stack size when twice the standard memory is available.  The stackSize must be changed consistently at both locations or Modules could import the value from Kernel (e.g. Kernel.stackSize.)
