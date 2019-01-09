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

### Modifying Display.Mod for other screen geometries

### Patching or modifying the Bootloader to provide more memory to the Oberon system

### Modifying Display.Mod to account for a different base address

