# Tech Note 003 - PCF and Unicode Fonts
## Modifying Oberon to support large fonts and Unicode Glyphs

Adapting Oberon to use Unicode for text requires: 
* changing the font subsystem so that can handle large fonts
* a source of fonts with good coverage of the Unicode space
* changing the text subsystem to handle the Unicode code points which may span several bytes in a text and which can have ordinal values that are sixteen bits in size or larger.

### On-demand loading of blocks of font data

Classic Oberon fonts are limited to a maximum of 128 characters and the in-memory structure keeps the glyph metrics and bitmaps in an array of memory contiguous with the font metadata. This is fine for small raster fonts. Unicode fonts may be larger than available memory in a risc system however, and most character data will not be accessed.

Rather than loading the whole font into memory at one time, the Font structure may be constructed instead to intitially load only basic font information (name, general dimensions) and then load ranges of specific character (codepoint) data when a codepoint within a range is requested.

Michael Schierl has provided some [patches](https://github.com/schierlm/OberonEmulator/blob/master/ProposedPatches/use-utf8-charset.patch) to Fonts.Mod for on-demand loading of font data, and also to other core modules for using Unicode code points in Oberon text.

The following structure from Michael's patches separates the font information from the codepoint and raster data. Upon initialization the T1 array is full of zeros and there are no raster blocks (`block` is `NIL`.)

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

When not `0` or `1`, `pa` is a pointer to a range of 64 values (base of `pa` indexed by `codepoint DIV 40H MOD 40H`) that also can be `0`, `1`, or a valid pointer to a raster block containing data for 64 codepoints. 

If a valid pointer then `pa` receives that pointer for further dereferencing.  With this new value, `pa` of `0` results in the default pattern, `1` causes the loading of raster data for this range of 64 codepoints and then dereferencing is retried, or `pa` is further dereferenced to obtain the range of raster data.

Finally, `pa` indexed by `codepoint MOD 40H` results in `patadr` which points three bytes into a multi-byte structure containing byte values of dx, x, y, w, and h, followed by the bitmap pattern to be displayed for that codepoint. 
 
```
                                                                        -3 [  dx ]
                                                                        -2 [  x  ]
T1:  0 [ ptr ]  pa: --->  0 [  1  ]                                     -1 [  y  ]
     1 [  1  ]            1 [ ptr ]  pa: ---> 0 [ ptr ] patadr: ------>  0 [  w  ]
     2 [  0  ]            2 [  1  ]           1 [  1  ]                  1 [  h  ]
      ...                  ...                2 [  1  ]     2 to (h*w/8)+2 [ bitmap ]
    48 [  0  ]           64 [  0  ]            ...
                                             64 [  1  ]
    
    
 ```   
By changing the exported FontDesc record and changing the pattern retrieval procedure (from GetPat to GetUniPat) in Fonts.Mod, the symbol file for the module also changes. Because the symbol file changes, any module that imports Fonts.Mod must also be recompiled before being used in a new system using the updated Fonts module. Essentially, all of the outer core must be recompiled before rebooting the Oberon system. In addition, any references to GetPat in other modules must be updated to GetUniPat or compilation will fail. Further changes to importing modules are described below in the Unicode Glyphs in Texts section.

The following commands in the Oberon system will recompile the Fonts module and dependent Oberon 'Outer Core' Modules:

```
ORP.Compile Fonts.Mod/s Texts.Mod/s ~
ORP.Compile Input.Mod/s Display.Mod/s Viewers.Mod/s ~
ORP.Compile Oberon.Mod/s MenuViewers.Mod/s Graphics.Mod/s ~
ORP.Compile GraphicFrames.Mod/s TextFrames.Mod/s System.Mod/s ~
ORP.Compile ORS.Mod/s ORB.Mod/s ORG.Mod/s ORP.Mod/s ~
ORP.Compile Edit.Mod/s Tools.Mod/s ORTool.Mod/s ~
ORP.Compile Draw.Mod/s ~
```

### Pcf to Raster Blocks

While Oberon font files contiguously store glyph metadata and glyph bitmaps, PCF font files keep them in separate tables. In addition, the PCF format stores the bitmap data bit-reversed from the Oberon font format.

A modified [Fonts.Mod](https://raw.githubusercontent.com/io-core/Edit/main/Fonts.Mod) loads Oberon font data or PCF font data conditionally on the format of the font as identified by the first several bytes of the file.

### Unicode Glyphs in Texts

Displaying Unicode text requires further changes to modules originally written to expect 7-bit character data. By replacing `c*: CHAR` with `codepoint*: INTEGER;` in the exported record definition of `scanner` in `Texts.Mod`, everything in Oberon that uses the scanner functionality is required to adapt to Unicode in order to function. Since UTF8 Unicode codepoints span a variable number of bytes, a `UnicodeWidth` procedure calculates the byte width of a codepoint, the `Pos` function is updated to adapt to the variable width of codepoints, and additional procedures `WriteUnicode`, `ReadUnicode`, and `ReadUnicodeRest` convert 32-bit codepoint values to and from UTF8 encoded byte sequences.

`Edit.Mod` gains an `InsertUnicode` function, the `DeleteChar` function in `GraphicFrames.Mod` is adapted to variable-length codepoints. The remainder of the changes primarily consist of changing `GetPat` to `GetUniPat`, changing comparisons of `c `(type `CHAR`) to comparisons of `codepoint` (type `INTEGER`), making calls to `Texts.ReadUnicode` instead of `Texts.Read` (and likewise for `Write`). Changes are required in [Texts.Mod](https://github.com/io-core/Edit/blob/main/Texts.Mod) [Draw.Mod](https://github.com/io-core/Draw/blob/main/Draw.Mod), [Edit.Mod](https://github.com/io-core/Edit/blob/main/Edit.Mod), [Fonts.Mod](https://github.com/io-core/Edit/blob/main/Fonts.Mod), [GraphicFrames.Mod](https://github.com/io-core/Draw/blob/main/GraphicFrames.Mod), [Graphics.Mod](https://github.com/io-core/Draw/blob/main/Graphics.Mod), [Input.Mod](https://github.com/io-core/Oberon/blob/main/Input.Mod), [Net.Mod](https://github.com/io-core/System/blob/main/Net.Mod), [Oberon.Mod](https://github.com/io-core/Oberon/blob/main/Oberon.Mod), [ORP.Mod](https://github.com/io-core/Build/blob/main/ORP.Mod), [System.Mod](https://github.com/io-core/System/blob/main/System.Mod), and [TextFrames.Mod](https://github.com/io-core/Edit/blob/main/TextFrames.Mod). 

The above changes merely allow the text subsystem of a modified Oberon to display more graphical characters than the original system. More changes are required to encompass the full Unicode specification, including e.g. combining characters, line breaking, right-to-left text, etc.

![font example](https://github.com/io-core/Technotes/blob/main/images/fonts.png "font example")
