# Tech Note 001
Changing OberonV5 to use NL instead of CR for terminating lines in text

In order to be able to work smoothly with existing files, an Oberon system that accepts NL to terminate lines should also accept CR but should emit NL when producing new line termination characters.

To adjust OberonV5, the following files are modified:
* ORS.Mod
* Edit.Mod
* GraphicFrames.Mod
* Input.Mod
* Texts.Mod
* TextFrames.Mod
* System.Mod


