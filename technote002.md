# Tech Note 002 - Adjusting Module, Stack, Heap, and Video Memory
## Adjusting OberonV5 memory use

The Oberon V5 system calculates its module, stack, heap, and memory locations based on two values delivered to it via the BootLoader: 

* `MemLim` is a 32-bit value placed at memory location `12` and is the constant `E7EF0H` in the original 2013 Project Oberon. 
* `stackOrg` is a 32-bit value placed at memory location `24` and is the constant `80000H` in the original 2013 Project Oberon.

The start of video memory is also a constant in the original 2013 Project Oberon: `E7F00H` which happens to be equivalent to MemLim + 16. The original Display.Mod in 2013 Project Oberon also hard-coded the screen geometry to a monocrome 1024x768 framebuffer.

The above configuration uses 1MB of ram for programs and video memory, giving 50% to module space and 50% to heap + display.

In OberonV5 the bootloader places the Oberon inner core binary image into the system RAM starting at address zero and then writes the MemLim and stackOrg values. The image includes the pre-linked inner core modules. 

Those modules then initialize the dynamic structures of Oberon including the Stack (which grows down from stackOrg) and the heap, which occupies the space between stackOrg and MemLim.

OberonV5 can be modified to use more memory for the display, the heap, and for module space by 1) Modifying Display.Mod for other screen geometries, 2) Patching or modifying the bootloader, and 3) Modifying Display.Mod to account for a different base address. 

### Modifying Display.Mod for dynamic screen geometries

Other geometries than 1024x768 can be supported by modifying only Display.Mod (and, of course, the hardware or emulator of the Oberon RISC system.) The simplest and most direct method is to adjust the constants in Display.Mod to reflect the changed width and height and base offset of the screen. 

If the screen resolution is not constant then Display.Mod needs to acquire the geometry at run-time. One method is to use a convention of placing the width of the screen at location `base+4`, the height of the screen at location `base+8` and a constant, e.g. `53697A66H` at the base of the screen buffer memory. Display.Mod may these values and if it finds the constant then it adopts the width and height values, otherwise it uses the default values of 1024x768.  Alternative methods might include reading hardware registers or obtaining values from the boot loader like is done with stackOrg and MemLim.

As the base offset for the screen has not changed, the code may still use the constant `E7F00` in its drawing calculations. The Display.Mod code can no longer use the constant `128` as the number of bytes that contain a row of pixels on the screen.
 An updated Display.Mod must calculate this at initialziation time.
 
### Patching or modifying the Bootloader to provide more memory to the Oberon system

### Modifying Display.Mod to account for a different base address

