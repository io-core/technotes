# Tech Note 001 - Newlines
## Changing OberonV5 to use NL instead of CR for terminating lines in text

In order to be able to work smoothly with existing files, an Oberon system that accepts NL to terminate lines should continue to accept CR in source texts but should emit NL when producing new line termination characters.

To adjust OberonV5, the following files require the following changes:
* ORS.Mod (already in latest Project Oberon version, older versions were more restrictive)
    * in `PROCEDURE HexString` use `WHILE ~R.eot & (ch <= " ") DO Texts.Read(R, ch) END ;`
* Graphics.Mod
    * in `PROCEDURE CopyCaption` use `Texts.Write(TW, 0AX); Texts.OpenReader(R, T, src(Caption).pos);`
    * in `PROCEDURE ReadCaption` use `BEGIN obj(Caption).pos := T.len + 1; Texts.Write(TW, 0AX);`
* GraphicFrames.Mod
    * in `PROCEDURE CaptionCopy` use   `BEGIN Texts.Write(W, 0AX);`
    * in `PROCEDURE NewCaption` use   `BEGIN Texts.Write(W, 0AX);`
* Texts.Mod
    * in `CONST` section use   `TAB = 9X; CR = 0DX; NL = 0AX;  maxD = 9;`
    * in `PROCEDURE Scan` use  `WHILE ~S.eot & (ch <= " ") DO`
    * in `PROCEDURE Scan` use  `IF (ch = NL) OR (ch = CR) THEN INC(S.line) END ;`
    * in `PROCEDURE WriteLn` use   `BEGIN Write(W, NL)`
* TextFrames.Mod
    * in `CONST` section use     `BS = 8X; TAB = 9X; CR = 0DX; NL = 0AX; DEL = 7FX;`
    * in `PROCEDURE DisplayLine` use     `WHILE (nextCh # NL) & (nextCh # CR) & (R.fnt # NIL) DO`
    * in `PROCEDURE Validate` use       `REPEAT Texts.Read(R, nextCh); INC(pos) UNTIL R.eot OR (nextCh = NL) OR (nextCh = CR)`
    * in `PROCEDURE Write` use     `ELSIF (20X <= ch) & (ch <= DEL) OR (ch = NL) OR (ch = CR) OR (ch = TAB) THEN`
* System.Mod
    * in `PROCEDURE Directory` use     `IF (ch = "^") OR (ch = 0DX) OR (ch = 0AX) THEN`
    
   
If the source texts (.Mod files) being introduced to an un-converted OberonV5 system already use newlines instead of carriage-returns and ORS is from an older, more restrictive version then ORS in the un-converted system must be modified first (and be unloaded so the new ORS module may be loaded) before compiling the rest of the source files, or the compiler will emit an error when attemptig to compile HexStrings in the source files.

Aside from ORS the above files do not have to be compiled in any particular order.

Since all of the above files are in the Outer Core of OberonV5 a new Inner Core does not have to be compiled or installed. A newline-friendly system can be produced merely by modifying and compiling the above modules and then restarting the system.

The above modifications do not account for the CR LF combination which more appropriately would be treated as a single line break rather than two as results with the above changes.
