# Tech Note 001
Changing OberonV5 to use NL instead of CR for terminating lines in text

In order to be able to work smoothly with existing files, an Oberon system that accepts NL to terminate lines should also accept CR but should emit NL when producing new line termination characters.

To adjust OberonV5, the following files are modified:
* ORS.Mod (in PROCEDURE HexString use WHILE ~R.eot & (ch <= " ") DO Texts.Read(R, ch) END ;  )
* GraphicFrames.Mod
* Input.Mod
* Texts.Mod
* TextFrames.Mod
* System.Mod

If the source texts (.Mod files) being introduced to an un-converted OberonV5 system already use newlines instead of carriage-returns then ORS in the un-converted system must be modified first (and be unloaded so the new ORS module may be loaded) before compiling the rest of the source files, or the compiler will emit an error when attemptig to compile HexStrings in the source files.


