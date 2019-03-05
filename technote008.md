# Tech Note 008 - Packages
### Packageing in Oberon with requirements and semantic versioning

It would be useful to have a package management system for Oberon code and artifacts that assists the programmer in handling dependencies and versions. It would be helpful if such a system were not tied to a particular distribution of Oberon but would allow a package (if conservatively coded) to be automatically obtained and integrated across implementations of the system.

Packaging goals:

* A collection of related modules may be packaged together for a unified release
* Modules may depend on minimum and maximum versions of other modules
* Tooling to assist in obtaining packages
* Compatibility across distributions
* Deterministic builds
* Signatures and verification

The original Project Oberon set of modules and resources (font files, etc.) can be viewed as one undifferentiated package, version 5 of the complete system, which does not require any other packages:

```
package [core]/System v5.0.0

requires ()
provides (
	Blink.Mod     FileDir.Mod        Kernel.Mod       ORC.Mod         RISC.Mod           Texts.Mod
	BootLoad.Mod  Files.Mod          MacroTool.Mod    ORG.Mod         RS232.Mod          Tools.Mod
	Checkers.Mod  Fonts.Mod          Math.Mod         ORP.Mod         SCC.Mod            Viewers.Mod
	Curves.Mod    GraphicFrames.Mod  MenuViewers.Mod  ORS.Mod         Sierpinski.Mod
	Display.Mod   Graphics.Mod       Modules.Mod      ORTool.Mod      SmallPrograms.Mod
	Draw.Mod      GraphTool.Mod      Net.Mod          PCLink1.Mod     Stars.Mod
	EBNF.Mod      Hilbert.Mod        Oberon.Mod       PIO.Mod         System.Mod
	Edit.Mod      Input.Mod          ORB.Mod          Rectangles.Mod  TextFrames.Mod
)
 ```

One way to divide up the modules is into the following packages: Build, Edit, fonts, Kernel, Modules, Paint, System, Draw   Files, Oberon. The package file `System.Pkg` which, when its requirements are also fetched, encompases the original Project Oberon files:

```
package [core]/System v5.1.0

requires (
        [core]/Kernel v5.0.0
        [core]/Files v5.1.0
        [core]/Modules v5.1.0
        [core]/Oberon v5.1.0
        [core]/Edit v5.1.0
        [core]/Draw v5.1.0
        [core]/Paint v1.0.0
)

provides (
	Blink.Mod
	RISC.Mod
	Tools.Mod
	BootLoad.Mod
	Net.Mod
	System.Mod        
	PCLink1.Mod
	RS232.Mod
	Math.Mod
	PIO.Mod
	SCC.Mod
	System.Tool
)

```

An Oberon application package may be compatible with both the io-core system and with the original Project Oberon 2013 system, as below:

```
package [extra]/Score v0.1.0

requires (
        [core]/System v5.0.0
)

provides (
        Notes.Mod
        Score.Mod
        TabFrames.Mod
        Tabs.Mod
)

```


