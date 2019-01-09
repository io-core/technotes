# Tech Note 002 - Adjusting Module, Stack, Heap, and Video Memory
### Adjusting OberonV5 memory use

The Oberon V5 system calculates its module, stack, heap, and memory locations based on two values delivered to it via the BootLoader: 

* `MemLim` is a 32-bit value placed at memory location `12` and is the constant `E7EF0H` in the original 2013 Project Oberon. 
* `stackOrg` is a 32-bit value placed at memory location `24` and is the constant `80000H` in the original 2013 Project Oberon.

The start of video memory is also a constant in the original 2013 Project Oberon: `E7F00H` which happens to be equivalent to MemLim + 16. The original Display.Mod in 2013 Project Oberon also hard-coded the screen geometry to a monocrome 1024x768 framebuffer.


* Modifying Display.Mod for other screen geometries
* Patching or modifying the Bootloader to provide more memory to the Oberon system
* Modifying Display.Mod to account for a different base address

