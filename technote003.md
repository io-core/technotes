# Tech Note 003 - PCF and Unicode Fonts
## Modifying the Oberon file system to support large fonts and Unicode

Adapting Oberon to use Unicode for text requires: 
* changing the font subsystem so that can handle large fonts
* a source of fonts with good coverage of the Unicode space
* changing the text subsystem to handle the Unicode code points which may span several bytes in a text and which can have ordinal values that are sixteen bits in size or larger.

### fonts to font chunks

Classic Oberon fonts are limited to a maximum of 128 characters, and the in-memory structure keeps the glyph metrics and bitmaps in an array of memory contiguous with the font metadata.

The Font structure could instead maintain a list of raster blocks of glyph data, each containing a subset of glyps. If a subset size of 64 were chosen, Classic Oberon fonts can be held in a font structure with two raster blocks.

The character size (e.g. 8, 16, or 32 bits) is exported, and changing it to support unicode (16 bits for each code plane, of which there are 16 so far) does require recompilation of all importing modules, which includes most of the Outer Core of Oberon.

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



### pcf to raster blocks chunks


### unicode in texts

