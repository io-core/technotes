# Tech Note 003 - PCF and Unicode Fonts
## Modifying the Oberon file system to support large fonts and Unicode

Adapting Oberon to use Unicode for text requires: 
* changing the font subsystem so that can handle large fonts
* a source of fonts with good coverage of the Unicode space
* changing the text subsystem to handle the Unicode code points which may span several bytes in a text and which can have ordinal values that are sixteen bits in size or larger.

### On-demand loading of blocks of font data

Classic Oberon fonts are limited to a maximum of 128 characters and the in-memory structure keeps the glyph metrics and bitmaps in an array of memory contiguous with the font metadata. This is fine for small raster fonts. Unicode fonts may be larger than available memory in a risc system however, and most character data will not be accessed.

Rather than loading the whole font into memory at one time, the Font structure may be constructed instead to intitially load only basic font information (name, general dimensions) and then load ranges of specific character (codepoint) data when a codepoint within a range is requested.

The following structure separates the font information from the codepoint and raster data. Upon initialization the T1 array is full of zeros and there are no raster blocks (`block` is `NIL`.)

```  
  TYPE Font* = POINTER TO FontDesc;
    RasterBlock = POINTER TO RasterBlockDesc;
    FontDesc* = RECORD
      name*: ARRAY 32 OF CHAR;
      height*, minX*, maxX*, minY*, maxY*: INTEGER;
      next*: Font;
      T1: ARRAY 48 OF INTEGER;
      block: RasterBlock;
    END;

    RasterBlockDesc = RECORD
      next: RasterBlock;
      offs: INTEGER;
      raster: ARRAY 1000 OF BYTE;
    END;

```
If a raster block contains data for 64 codepoints, and an indirection table refers to 64 raster blocks, then a `T1` array of 16 pointers to indirection tables can cover the 65536 codepoints of the Basic Multilingual Plane of Unicode. Unicode defines 17 planes so far however, although planes 3-13 don't have any characters assigned yet, plane 14 currently contains non-graphical characters, and planes 15 and 16 are private-use planes. If Unicode use in Oberon is limited to planes 0-2 then a `T1` array of 48 pointers can refer to all of the graphical characters defined in Unicode 11.0

Most fonts will contain glyphs for only a subset of the Unicode codepoints. When a font does contain glyphs for a large number of codepoints, few of those codepoints will actually be displayed at any one time.

To find the raster pattern for a codepoint the system first examines `<font>.T1[codepoint DIV 1000H]` which will either contain a pointer `pa` to an intermediate structure representing 4096 codepoints or the value `0` which means the font does not contain that range of codepoints and a default pattern is returned, or the value `1` indicating that data for that range has not been loaded yet, which triggers an attempt to load raster data from the font file in order to satisfy the request. 

When not zero or one, `pa` is a pointer to a range of 64 values (base of `pa` indexed by `codepoint DIV 40H MOD 40H`) that also can be `0`, `1`, or a valid pointer `pb` to a raster block containing data for 64 codepoints. In this case zero results in the default pattern, one causes the loading of raster data for this range of 64 codepoints, or `pb` is dereferenced to obtain the range of raster data.





The character size (e.g. 8, 16, or 32 bits) is exported, and changing it to support unicode (16 bits for each code plane, of which there are 16 so far) does require recompilation of all importing modules, which includes most of the Outer Core of Oberon.


### pcf to raster blocks chunks


### unicode in texts

