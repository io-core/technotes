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
      T1: ARRAY 16 OF INTEGER;
      block: RasterBlock;
    END;

    RasterBlockDesc = RECORD
      next: RasterBlock;
      offs: INTEGER;
      raster: ARRAY 1000 OF BYTE;
    END;

```

The character size (e.g. 8, 16, or 32 bits) is exported, and changing it to support unicode (16 bits for each code plane, of which there are 16 so far) does require recompilation of all importing modules, which includes most of the Outer Core of Oberon.


### pcf to raster blocks chunks


### unicode in texts

