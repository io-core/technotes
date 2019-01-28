# Tech Note 003 - PCF and Unicode Fonts
## Modifying the Oberon file system to support large fonts and Unicode

Adapting Oberon to use Unicode for text requires changing the font subsystem so that can handle large fonts, a source of fonts with good coverage of the Unicode space, and changing the text subsystem to handle the Unicode code points which may span several bytes in a text and which can have ordinal values that are sixteen bits in size or larger.

### fonts to font chunks

Classic Oberon fonts are limited to a maximum of 128 characters, and the in-memory structure keeps the glyph metrics and bitmaps in a single 'chunk' or array of memory contiguous with the font metadata.

The Font structure could instead maintain a list of 'chunks' of glyph data, each containing 128 glyps, with the codepoint DIV 80H identifying the chunk and the codepoint MOD 80H identifying the glyph. Classic Oberon fonts can be held in a font structure with a single chunk. Chunks could be dynamically loaded as requests for glyphs on not-yet-loaded chunks arrive. The number 128 of glyphs is arbitrary, the chunk size is not exported to client modules.

The character size (e.g. 8, 16, or 32 bits) is exported, and changing it to support unicode (16 bits for each code plane, of which there are 16 so far) does require recompilation of all importing modules, which includes most of the Outer Core of Oberon.

`  TYPE
    Font* = POINTER TO FontDesc;
    Chunk = POINTER TO ChunkDesc;

    FontDesc* = RECORD
      name*: ARRAY 32 OF CHAR;
      height*, minX*, maxX*, minY*, maxY*: INTEGER;
      next*: Font;
      chunk0: Chunk;
    END ;

    ChunkDesc = RECORD
      next: Chunk;
      index: INTEGER;
      T: ARRAY 128 OF INTEGER;
      raster: ARRAY 2360 OF BYTE;
    END;

    LargeChunkDesc = RECORD (ChunkDesc) ext: ARRAY 6656 OF BYTE END ;
    LargeChunk = POINTER TO LargeChunkDesc;
    
PROCEDURE GetUniPat*(fnt: Font; codepoint: INTEGER; VAR dx, x, y, w, h, patadr: INTEGER);
  VAR chunk, index, pa: INTEGER;  dxb, xb, yb, wb, hb: BYTE;
BEGIN
  chunk := codepoint DIV 80H;
  index := codepoint MOD 80H;
  
  (* TODO: indexing to appropriate chunk goes here *)
  
  pa := fnt.chunk0.T[index]; patadr := pa;
  SYSTEM.GET(pa-3, dxb); SYSTEM.GET(pa-2, xb); SYSTEM.GET(pa-1, yb); SYSTEM.GET(pa, wb); SYSTEM.GET(pa+1, hb);
  dx := dxb; x := xb; y := yb; w := wb; h := hb;
  IF yb < 128 THEN y := yb ELSE y := yb - 256 END
END GetUniPat;`

### pcf to font chunks


### unicode in texts

